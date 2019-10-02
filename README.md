# Strict CSP for Everyone!

## A Problem

XSS is bad. _[Insert statistics here]_. New and exciting APIs make a bad thing worse by enhancing the
consequences of injection. We think we have a good handle on a deployable approach to CSP. _[Insert
discussion of [strict CSP](https://csp.withgoogle.com/docs/strict-csp.html) here]. What if we gently
suggested that all developers follow that pattern by requiring developers to opt-out of its
protections?

## A Proposal

Let's require pages to assert a Content Security Policy of some sort by enforcing a Strict CSP variant
by default, and allowing servers to override that enforcement with their own custom policy if they
choose to do so.

This might look like the following:

1.  For each navigation request, the user agent generates a nonce, and appends it to the request as an
    HTTP header. For example, `Sec-Script-Nonce: "nonce-bct4F1nh9M+zaoRtQm+lbg"`.

2.  If the server's response does not contain a `Content-Security-Policy` header, the user agent will
    inject something like the following header:

	  ```
    Content-Security-Policy: script-src [NONCE] 'strict-dynamic';
	                         base-uri 'self';
	                         object-src 'none';
    ```
    
    _Note that we can assume support for modern CSP here, so we don't need the backwards-compatibility cruft that Strict CSP recommends._

Developers would have a few options:

1.  Folks who aren't paying attention will not be able to execute script. In the best case, this will
    encourage them to pay attention (at which point they'll fall into bucket 2 or 4 below). In the worst
    case, they won't be able to execute script (which would be unfortunate).
    
2.  Folks who are paying attention, but who haven't deployed CSP will yell at me on Twitter. Then they
    can either deploy a custom policy that's more suitable for their specific needs. This might be an
    opt-out policy (e.g. `Content-Security-Policy: default-src * 'unsafe-inline' 'unsafe-eval'`, or a
    CSP of their own (perhaps hash- or allowlist based rather than nonce-based).
    
    For simplicity in the first case, we could even create an explicit opt-out (e.g. `I-Dont-Like: CSP`)
    instead.

3.  Folks who are already delivering a `Content-Security-Policy` header will continue doing so, and won't
    have to change anything about what they're doing.

4.  Folks who are paying attention, and who are willing to accept the default policy, can accept the
    nonce the browser has suggested and inject it into their script tags as appropriate, or they can
    craft a custom policy that's more suitable for their specific needs.

## FAQ

### Isn't this a bad idea?

No. It's a good idea.

### Really? How would you possibly deploy it?

It's not something we can turn on tomorrow, but it doesn't seem impossible to get out into the world. I
can imagine a phased rollout with a few ratchet points:

*   We can begin warning in the console that no CSP is present.

*   When we start sending the generated nonce, we can set up a report-only policy rather than an enforced
    policy. That will flood the console with warnings rather than breaking pages.

*   We can first require enforcement only when using new and exciting APIs that increase the risks XSS
    presents. If you want native filesystem access, for instance, perhaps you need a CSP of some sort.

*   Then we go all in, and solve XSS, once and for all.

### Do you really want to make everyone learn CSP's weird syntax?

No. Not really. The advantage of requiring CSP is that it exists. The disadvantage is that it's incredibly
complicated and overly broad. Default enforcement might be a good enough justification to take another
look at something like [`Scripting-Policy`](https://github.com/mikewest/csp-next#xss-mitigation) instead,
which would substantially simplify the model.
