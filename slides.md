## Mozilla Persona
### For Good and For Awesome

- `https://github.com/mozilla/browserid`

----------------------------------------

**Jed Parsons**, Engineer, Mozilla

- `jparsons@mozilla.com`
- `https://github.com/jedp`
- `@drainmice`

**DevCon5**, New York, 2013

## What is Persona?

**Persona** is a beautiful way to sign into websites without passwords.

Users simply need to verify an email address once and then they can sign in
anywhere with just two clicks.

Persona works on all modern browsers and is easy for developers to incorporate
into their websites.

## What does it Look Like?

- On desktop?
- On mobile?

Some examples:

- bornthiswayfoundation.org
- sloblog.io
- crossword.thetimes.co.uk
- voo.st
- mineshafter.info
- reasonwell.com

## What are they saying?

> You can't run a FB-only login system without alienating a
   significant chunk of most audiences. ... Now that Persona and FB
   auth are on equal footings, we have yet to have anyone complain
   about the signup process.

Source: [http://news.ycombinator.com/item?id=4599881](http://news.ycombinator.com/item?id=4599881)

## What are they saying?

Eran Hammer, former lead author and editor of OAuth

> Mozilla Persona is great not only for the right vision but for the right
> leadership. Putting to shame all previous web identity attempts.

- https://twitter.com/eranhammer/status/335558480285011969

## First, Review

Typical sign-in flow today:

- Provide an email address
- Create a password
- Wait for a confirmation email
- Click the link in the email
- Use the password for future sign-ins

Hmm ... 

## Mozilla Persona Flow

First time flow:

1. Click login
2. **Enter an email address**
3. **Authenticate with your email provider**

Second time flow:

1. Click login, if you're not automatically logged in
2. **Select an email address**

Done in two clicks.  No passwords.  No kittens were harmed.

## Why Email Addresses

- People know them
- They make sense: name@place
- Natural for capturing contexts: me@work, me@home, etc.
- You can have multiple identities
- Provide developers a direct means of contacting users
- Most sites want email anyway; one less step for signup

## Benefits for Developers

- There are **no passwords**
- Storing passwords is a liability
- Conversion rate
- No lock-in; Email addresses are universal

## In your Website

    <script src="https://login.persona.org/include.js"></script>

## A Complete Example

    navigator.id.watch({













    });

## Maybe provide the logged-in user

    navigator.id.watch({
      loggedInUser: #{ whoYouThinkIsLoggedIn },












    });

## Provide an onlogin callback

    navigator.id.watch({
      loggedInUser: #{ whoYouThinkIsLoggedIn },

      onlogin: function(assertion) {





      },




    });

## Provide an onlogout callback

    navigator.id.watch({
      loggedInUser: #{ whoYouThinkIsLoggedIn },

      onlogin: function(assertion) {





      },

      onlogout: function() {
        window.location = "/logout";
      }
    });


## Request an identity

    navigator.id.request();

For example:

    $('#login-button').click(function() {
      navigator.id.request();
    });


## Handling a login

    navigator.id.watch({
      loggedInUser: #{ whoYouThinkIsLoggedIn },

      onlogin: function(assertion) {
        $.post('/login',
               {assertion: assertion},
               function(data) {
                 window.location = "/home";
               }
      },

      onlogout: function() {
        window.location = "/logout";
      }
    });

## Verifying identity ownership

On your server:

    var body = qs.stringify({
      assertion: assertion,
      audience: YOUR_SITE
    });
    var options = {
      uri: 'https://verifier.login.persona.org/verify',
      headers: {
        'content-type': 'application/x-www-form-urlencoded',
        'content-length': body.length
      }
    };
    request.post(options, function(err, res, body) {
      if (err) return  callback(err);
      return callback(null, JSON.parse(body));
    });

Use our verifier, or host your own, which is faster.

## Verifier Return Values

The verifier will return JSON like so:

    {
      "status": "okay",
      "email": "shakti@idomene.us",
      "audience": "https://hereswaldo.com",
      "expires": 1354217396705,
      "issuer": "idomene.us"
    }

If the assertion was not good, the `status` will not be `okay`.

## logout

    navigator.id.logout();

## summary

Websites just do this:

1. Load `https://login.persona.org/include.js`
2. Setup `onlogin` and `onlogout` callbacks
3. Verify the assertion (proof of ownership)

## Security Considerations

[https://developer.mozilla.org/en-US/docs/Persona/Security_Considerations](https://developer.mozilla.org/en-US/docs/Persona/Security_Considerations)

- Verify assertions on your server
- Specify the `audience` parameter to the verifier
- Use HTTP libraries that verify SSL certs
- Implement CSRF (Cross-Site Request Forgery) protection
- Implement CSP (Content Security Policy)

## some extra details

    navigator.id.request({
      oncancel: function() {
        // revert to previous state ...
      },

      siteName: "Extraordinary Cats",
      siteLogo: "/img/logo.png",
      backgroudColor: "#123456",
    });

Note: https required for site logo, though we are investigating data urls.

## What's really happening here?

Let's nerd out for a moment.

How does the protocol work?

Dramatis Personae:

- Shakti the Shopper
- Waldo the Website, merchant of striped clothing
- Idomeneus the Identity Provider

## 1. Identity Provisioning

Wherein Shakti proves her identity

1. Shakti wants to login to `hereswaldo.com` as `shakti@idomene.us`
2. Shakti generates a public and private keypair
3. Shakti must prove she is actually `shakti@idomene.us`
4. Shakti is directed to Idomeneus
5. Idomeneus asks Shakti to authenticate
6. She does, so Idomeneus asks Shakti for her public key
7. Idomeneus constructs a JWT with email address, public key, and expiry date
8. Idomeneus signs the JWT with his own private key
9. Idomeneus returns this **signed certificate** to Shakti
10. Shakti stores this as proof of her `shakti@idomene.us` identity
11. Shakti's UA stores the **private key** she generated along with the certificate

## 2. Assertion Generation

Wherein Shakti declares she wants to sign in

1. Shakti makes a JWT containing her email, Waldo's address, and a short expiry
2. Shakti signs this JWT with her private key
3. This is a **signed assertion** of identity
4. She pins a copy of her `idomene.us` certificate to her assertion
5. This is a **backed assertion** (backed by the certificate)
6. Shakti hands the backed assertion to Waldo, the website

## 3. Assertion Verification

Wherein Waldo confirms Shakti's ownership of the identity

1. Waldo transmits the assertion down to his servers for handling
2. On the server, he checks both expiry dates
3. Waldo extracts the hostname from the email address (`idomene.us`)
4. Waldo asks `idomene.us` for its public key
5. Waldo uses the key to verify the signature on the certificate
6. With Shakti's public key from the cert, Waldo verifies the assertion signature
7. He now knows that Shakti is indeed `shakti@idomene.us`
8. He opens the doors to his shop and offers to sell Shakti a striped pullover

## Waldo's code?

    navigator.id.watch({
      onlogin: function(assertion) {
        $.post('/login', {assertion: assertion},
               function(data) {
                 window.location = "/home";
               }
      },
      onlogout: function() {
        window.location = "/logout";
      }
    });

    $('#signin').click(function() {
      navigator.id.request({
         oncancel: backToNormal,
         siteName: "Here's Waldo",
         siteLogo: "/img/hereswaldo.png"
       });
    });


## integrating with LDAP

- Mozilla IDP: `https://github.com/mozilla/vinz-clortho/`

## idp API

Anybody can be an identity provider.  You can be your own IdP!

How an identity provider shows that it speaks browserid:

`https://eyedee.me/.well-known/browserid`

    {
      "public-key": {
        "algorithm": "RS",
        "n": "876131 ...",
        "e": "65537"
      },
      "authentication": "/browserid/sign_in.html",
      "provisioning": "/browserid/provision.html"
    }

## what does persona do with this?

- check for your `/.well-known/browserid`
- if it exists, try your `provisioning` url
- if necessary, call your `authentication` url and then provision again
- If you don't have a `well-known` file, fallback to mozilla persona.org

This way the system is fully federated from the get-go, but still falls
back to `persona.org` if necessary in order to bootstrap the web to 
Persona.


## Some IdP Examples

- Mozilla IdP (vinz-clortho)
- sendmypin.org
- gno.mn
- eyedee.me

See `https://developer.mozilla.org/en-US/docs/Mozilla/Persona`

## Embedding

All you need is a sandboxed iframe

- make a sandboxed iframe
- source visible UI or inivisible js scripts:
    - https://login.persona.org/sign_in#NATIVE
    - https://login.persona.org/communication_iframe
- probably inject framescript to communicate between chrome and content

See the `internal_api` docs at `https://github.com/mozilla/browserid`

## Thanks

- `https://github.com/mozilla/browserid`

----------------------------------------

**Jed Parsons**, Engineer, Mozilla

- `jparsons@mozilla.com`
- `https://github.com/jedp`
- `@drainmice`

**DevCon5**, New York, 2013

