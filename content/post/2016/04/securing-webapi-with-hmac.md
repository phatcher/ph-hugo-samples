---
title: "Securing WebAPI with HMAC - Protocol"
date: 2016-04-20 14:00:00Z
draft: false
aliases:
- /post/securing-webapi-with-hmac-–-protocol/
categories:
- Security
- Algorithms
---
I'm working on a project where we are using [Azure B2C Active Directory](https://azure.microsoft.com/en-gb/services/active-directory-b2c/)
 to provide authentication so that we can avoid the security headaches associated with managing a password store for it.
 
Unfortunately, the current implementation doesn't support .NET Windows applications or standalone services where no user-interface 
is present - it will eventually, but I can't wait on the eventual delivery date for these two use-cases.  I don't want to re-introduce 
user name/password for this, as I'd introduce the same security issues that I was trying to avoid in the first place, but following a
 bit of research I decided the way forward was [HMAC authentication](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code).
 
I did have a look around the net, but I couldn't find an implementation that had a NuGet package and was fully tested, so I wrote my own based 
on the code [here](https://bitbucket.org/pwalat/piotr.webapihmacauth)
 
The flow from the client side looks like this..
1. Retrieve your client id and secret from where you are keeping it on the client
1. Produce a message representation 
1. Hash it with the secret
1. Send the message

Server side, the flow is…
1. Check if the message is HMAC authorized, if not ignore it
1. Check if the message is within time e.g. +/- 5 minutes to allow for clock skew between client and server
1. Retrieve the client id header and use that to locate the secrets
1. Check the MD5 hash of the content.
1. Produce a message representation
1. Hash it with the secret
1. Check against the header hash If they match, check if we've seen if before, if so reject otherwise accept - this handles replay attacks.

As this is intended to be production quality code, I've constructed the framework with interfaces so that each of the steps are replaceable or extensible from the default implementation.
 
So the key interfaces are -

* **ISecretRepository** - Repository for client secrets.
* **IMessageRepresentationBuilder** - Responsible for constructing a canonical representation of a message.#
* **ISignatureCalculator** - Calculates a message signature
* **ISignatureValidator** - Validates a message Signature

ISecretRepository will vary widely for each implementation as server-side you will most likely being picking this up from your
database but on the client you might just acquire it from a config file or secure storage etc, so there's only a trivial 
implementation included in the library which uses a ConcurrentDictionary.
 
## Message Representation
How we construct the message representation is slightly different other implementations that I've seen.

```csharp
/// <summary>
/// Builds a canonical representation of the request messsage.
/// </summary>
public class MessageRepresentationBuilder : IMessageRepresentationBuilder
{
    private static readonly ILog Logger = LogProvider.GetLogger(MethodBase.GetCurrentMethod().DeclaringType);

    /// <summary>
    /// Construct a new instance of the  <see cref="MessageRepresentationBuilder"/> class.
    /// </summary>
    public MessageRepresentationBuilder()
    {
        // Function order matches required order in representation
        Headers = new List <Func <HttpRequestMessage, string>>
        {
            // NB Needed as the server-side request uri has been escaped e.g. /odata/%metadata
            // Also different .NET libraries encode differently - see https://stackoverflow.com/questions/575440/url-encoding-using-c-sharp/21771206#21771206
            m => WebUtility.UrlDecode(m.RequestUri.AbsolutePath.ToLower()),
            m => m.Method.Method,
            m => m.Content.Md5Base64(),
            m => m.Headers.MessageDate(),
        };
    }

    /// <summary>
    /// Headers in order required by the representation.
    /// </summary>
    protected List <Func <HttpRequestMessage, string>> Headers { get; private set; }

    /// <summary>
    /// Builds message representation as follows:
    /// HTTPMethod\n +
    /// Request URI (AbsolutePath i.e. excluding host, port and query)\n +
    /// Content-MD5\n +  
    /// Timestamp\n +
    /// </summary>
    /// <returns></returns>
    public string BuildRequestRepresentation(HttpRequestMessage requestMessage)
    {
        var values = new List <string>();
        foreach (var gm in Headers)
        {
           var value = gm(requestMessage);
           if (value == null)
           {
               // Fail on first null
               return null;
           }
           values.Add(value);
        }

        var result = string.Join("\n", values);

        Logger.DebugFormat("Representation: {0}", result);

        return result;
    }
}
```

For the server to rely on the information in the message, it needs to be part of hash, otherwise some party can replace that part 
of the message and replay the message and the server won't know that there's an issue.

All of the content is already handled by the hash, so we need some generic way of saying we want to include other message headers 
as part of the signature calculation and that's where the “x-msec-headers” header is used. It is a header containing the names of other headers that need to be included in the hash, for example a nonce value so that every request issued from a client is unique.

The nice thing about this is that the server implementation does not need to care - all of the information is held in the message 
itself; this idea is used to secure the [Amazon Simple Storage APIs](http://docs.aws.amazon.com/AmazonS3/latest/dev/RESTAuthentication.html#RESTAuthenticationConstructingCanonicalizedAmzHeaders), which is where I got the idea from.

## Signature Calculation

The next step is calculating the signature

```csharp
/// <summary>
/// Computes a HMAC signature.
/// </summary>
public class HmacSignatureCalculator : ISignatureCalculator
{
    private static readonly ILog Logger = LogProvider.GetLogger(MethodBase.GetCurrentMethod().DeclaringType);

    // TODO: Inject this via either constructor 
    private const string HmacScheme = "SHA256";


    /// <copydoc cref="ISignatureCalculator.Signature" />
    public string Signature(string secret, string value)
    {
        if (Logger.IsDebugEnabled())
        {
            Logger.DebugFormat("Source {0}: '{1}'", secret.Substring(0, 2) + "...", value);
        }

        var secretBytes = Encoding.Unicode.GetBytes(secret);
        var valueBytes = Encoding.Unicode.GetBytes(value);
        string signature;

        if (Logger.IsDebugEnabled())
        {
            Logger.DebugFormat("Value '{0}'", BitConverter.ToString(valueBytes));
        }
        
        using (var hmac = HmacProvider(secretBytes, HmacScheme))
        {
            var hash = hmac.ComputeHash(valueBytes);
            signature = Convert.ToBase64String(hash);
        }

        if (Logger.IsDebugEnabled())
        {
            Logger.DebugFormat("Signature {0} : {1}", HmacScheme, signature);
        }

        return signature;
    }

    private HMAC HmacProvider(byte[] secret, string value)
    {
        switch (value)
        {
            case "MD5":
            case "SHA1":
                throw new NotSupportedException(string.Format("Hash '{0}' is not secure and is not supported", value));

            case "SHA384":
                return new HMACSHA384(secret);

            case "SHA512":
                return new HMACSHA512(secret);

            default:
                return new HMACSHA256(secret);
         }
    }
}
```

So this allows us to change the encryption algorithm without affecting the rest of the system, bear in mind it wasn't that 
long ago when SHA-1 and MD-5 were acceptable ways of computing a one way hash.

## Signature Validation

The final part server side is validating the signature

```csharp
/// <summary>
/// Validates a HMAC signature if present.
/// </summary>
public class HmacSignatureValidator : ISignatureValidator
{
    private static readonly ILog Logger = LogProvider.GetLogger(MethodBase.GetCurrentMethod().DeclaringType);
    private const string CacheRegion = "hmac";

    private readonly ISignatureCalculator signatureCalculator;
    private readonly IMessageRepresentationBuilder representationBuilder;
    private readonly ISecretRepository secretRepository;
    private readonly ICache objectCache;

    /// <summary>
    /// Creates a new instance of the  <see cref="HmacSignatureValidator"/> class.
    /// </summary>
    /// <param name="signatureCalculator"> </param>
    /// <param name="representationBuilder"> </param>
    /// <param name="secretRepository"> </param>
    /// <param name="objectCache"> </param>
    /// <param name="validityPeriod"> </param>
    /// <param name="clockDrift"> </param>
    public HmacSignatureValidator(ISignatureCalculator signatureCalculator, 
        IMessageRepresentationBuilder representationBuilder,
        ISecretRepository secretRepository,
        ICache objectCache,
        int validityPeriod,            
        int clockDrift)
    {
        this.secretRepository = secretRepository;
        this.representationBuilder = representationBuilder;
        this.signatureCalculator = signatureCalculator;
        this.objectCache = objectCache;
        ValidityPeriod = validityPeriod;
        ClockDrift = clockDrift;
    }

    /// <summary>
    /// Get the allowable clock drift between server and client in minutes.
    /// </summary>
    public int ClockDrift { get; }

    /// <summary>
    /// Gets the validity period of a signature in minutes, used to avoid replay attacks.
    /// </summary>
    public int ValidityPeriod { get; }

    /// <summary>
    /// Validates whether we are using HMAC and if so is the signature valid.
    /// </summary>
    /// <param name="request"> </param>
    /// <returns> </returns>
    /// <remarks>
    /// We log all failures a Info, except for replay attacks which are logged as Warn as they may be a symptom of an attack.
    /// </remarks>
    public async Task <bool> IsValid(HttpRequestMessage request)
    {
        // TODO: Generalize so we can using any HMAC scheme.
        if (request.Headers.Authorization == null || request.Headers.Authorization.Scheme != HmacAuthentication.AuthenticationScheme)
        {
            // No authorization or not our authorization schema, so no 
            Logger.InfoFormat("Not our authorization schema: {0}", request.RequestUri);
            return false;
        }

        var isDateValid = request.IsMessageDateValid(ClockDrift);
        if (!isDateValid)
        {
            // Date is not present or valid
            Logger.InfoFormat("Invalid date: {0}", request.RequestUri);
            return false;
        }

        var userName = request.Headers.GetValues <string>(HmacAuthentication.ClientIdHeader).FirstOrDefault();
        if (string.IsNullOrEmpty(userName))
        {
            // No user name
            Logger.InfoFormat("No client id: {0}", request.RequestUri);
            return false;
        }

        var secret = secretRepository.ClientSecret(userName);
        if (secret == null)
        {
            // Can't find a secret for the user, so no
            Logger.InfoFormat("No secret for client id {0}: {1}", userName, request.RequestUri);
            return false;
        }

        if (!await request.Content.IsMd5Valid())
        {
            // MD5 is invalid, so no
            Logger.InfoFormat("Invalid MD5 hash: {0}", request.RequestUri);
            return false;
        }

        // Construct the representation
        var representation = representationBuilder.BuildRequestRepresentation(request);
        if (representation == null)
        {
            // Something broken in the representation, so no
            Logger.InfoFormat("Invalid canonical representation: {0}", request.RequestUri);
            return false;
        }

        // Compute the signature
        // TODO: Pass the encryption algorithm used e.g. SHA256
        var signature = signatureCalculator.Signature(secret, representation);

        // Have we seen it before
        if (objectCache.Contains(signature))
        {
            // Already seen, so no to avoid replay attack
            Logger.WarnFormat("Request replayed {0}: {1}", signature, request.RequestUri);
            return false;
        }

        // Validate the signature
        var result = request.Headers.Authorization.Parameter == signature;
        if (!result)
        {
            // Signatures differ, so no
            Logger.InfoFormat("Signatures differ {0}: {1}", signature, request.RequestUri);
        }
        else 
        {
            // Store valid signatures to avoid replay attack
            objectCache.Set(signature, userName, DateTimeOffset.UtcNow.AddMinutes(ValidityPeriod), CacheRegion);
        }

        // Return the signature validation.
        return result;
     }
}
```

Note that we log failures for debugging purposes but don't return that information back to the caller - they get a simple 
pass/fail to avoid exposing information to a potential attacker.

In part 2 I'll cover off how to use these to secure a WebApi controller, how to wire up the various components and how a 
client application might look.

There are a couple of points to remember with HMAC

* You still have a key distribution problem, how are you going to assign the client id/secret pairs 
* You should not use the client id for **authorization**, just **authentication** to make life easier when you invalidate the client id and/or assign multiple ids to the same user.

## Bibliography

Here are the various implementations I found around the net.

* [HMAC Authentication in Asp.Net Web API](https://www.piotrwalat.net/hmac-authentication-in-asp-net-web-api/) – this was my starting point (link seems to be down but article is also [here](http://www.voidcn.com/blog/lglgsy456/article/p-2321064.html))
* [Secure ASP.NET WebAPI authentication using API Key Authentication - HMAC Authentication](http://bitoftech.net/2014/12/15/secure-asp-net-web-api-using-api-key-authentication-hmac-authentication) – Taiseer Joudeh
* [Learn to Secure an ASP.NET Web API using HMAC](http://www.codeguru.com/csharp/.net/net_asp/learn-to-secure-an-asp.net-web-api-using-hmac.html) – Arun Karthick
* [How to secure an ASP.NET Web API](http://stackoverflow.com/questions/11775594/how-to-secure-an-asp-net-web-api/11782361) – Stackoverflow.
