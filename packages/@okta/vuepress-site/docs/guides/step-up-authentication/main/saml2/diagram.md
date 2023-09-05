<div class="three-quarter">

![Flow diagram that displays the back and forth between the Okta the Identity Provider, the user agent, and the Service Provider](/img/auth/step-up-authentication-acr-flowSAML.png)

</div>

At a high level, this flow has the following steps:

1. Per your use case, include the `acr_values` predefined parameter value in the authentication request.
2. The SP generates the SAML assertion and sends it in a SAML request to the browser.
3. The browser relays the SAML request to Okta.
4. Okta performs the authentication scenarios required in accordance with the predefined `acr_values` parameter value used in the SAML authentication request.
5. Okta generates and sends the SAML assertion response to the browser. The assertion contains the `acr` value from the request passed as `AuthnContextClassRef`. The factors (Authentication Method Reference (AMR) claims) used to authenticate the user are passed as a list of `AuthnContextDecl` in the `AuthnContext`.
6. The browser relays the SAML assertion to the SP.
7. The SP sends the `securitycontext` to the browser.
8. The browser requests access to the resource from the SP.
9. The SP responds with the requested resource.

<!-- @startuml Source for image. Generated using http://www.plantuml.com/plantuml/uml/

skinparam monochrome true
participant "Browser (User Agent)" as browser
participant "Service Provider" as sp
participant "Okta (Identity Provider)" as okta

autonumber "<b>#."
browser -> sp: Sends auth request with `acr_values` predefined parameter value
sp -> browser: Generates SAML assertion, sends SAML request
browser -> okta: Relays SAML request to Okta
okta <-> browser: Performs required authn per `acr_values` parameter value
okta -> browser: Generates, sends SAML assertion with `acr` value and factors
browser -> sp: Relays SAML assertion to SP
sp -> browser: Sends `securitycontext`
browser -> sp: Requests access to the resource
sp -> browser: Responds with requested resource
@enduml
-->