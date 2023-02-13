# Identity & Access Management meets DevOps & Agile

This article will dicuss the challenges of central IAM solutions in corporations when it faces the mostly contradictive desires of development teams for their applications. 

### My Background
I worked years in the IAM business consulting and implenting comprehensive solutions for authorization for clients. I also worked years as a product owner on the other side of the aisle and focussed on the flow of the developers to get stuff done. 

# What is Authentication and Authorization about?
Long story short - Authentication and Authorization is answering the question who should have access for what resources under what circumstances and for what reasons. 

**Authentication** for its part does answer if the user who is trying to get access to a resource is the user supposed to be. Here we are talking about buzzwords like SSO, MFA and Passwordless and such things and this article will never speak of it :) In todays world the result of an authentication mostly results in a user access token. 

To keep it simple for this article we call the instance of handling the authentication the Identity Provier (IdP) even though it is mostly split between the IdP managing the user and the login provider (mostly SSO for single sign-on). The Identity Provider will issue the user access token after a successfull authentication of the user. 

**Authorization** as the other part of the equation does answer which user has access to what resource. This can be as simple as whitelisting users in applications directly and as complex as a central sophisticated solution which decides in real time if a user has the right to access a resource.  

# Authorization Methods

This chapter will discuss the different types of Authorization from the two perspectives 
- **Application Team** which consists of developer and product owner and aiming for continous and fast flow of delivered features 
- **CISO** as representative IT security officer of a company who is in charge of compliance and governance in IT  

**Disclaimer**: All introduced access control methods below are showing the basics and just a selection of different kinds of implementation. It is more a selection of how these things work and they can also be implemted combined, more central or decentral. 

## Role Based Access Control (RBAC)
Role based access control is one of the most common access controls and works in two ways. 

First a user needs a role assignement in a central role management service which is integrated into the IDP and the user access token which is issues for a login does contain all assigned roles for a user. 

Second is the local access control logic in the application the user wants to access resources with. An example you can see below when the resource is an admin panel users want to navigate to. 

```python
import jwt

# The secret key used to sign the JWT
SECRET_KEY = "secret_key"

# The resource that the user is trying to access
RESOURCE = "admin_panel"

# The user's JWT token
TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJyb2xlIjoiYWRtaW4ifQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"

# Decode the JWT token
try:
    payload = jwt.decode(TOKEN, SECRET_KEY, algorithms=["HS256"])
except jwt.ExpiredSignatureError:
    print("Token expired.")
except jwt.InvalidTokenError:
    print("Invalid token.")

# Check the user's role
if payload["role"] == "admin":
    # Give the user access to the resource
    print("Access granted to resource: " + RESOURCE)
else:
    # Deny the user access to the resource
    print("Access denied to resource: " + RESOURCE)
``` 
The last part does check if the user has the role assigned and the access to a specific resource can be granted. 

**Application Team**: *It is easy to implement and maintain. We are in charge of the authorization logic and are not relying on third parties after we get the user access token. When we want or need to add a new authorization level we just add the next case to the logic and be done with this. As long as we find all information in the token we are good to go and hopefully our access control does not get to complex.*

- ownership (locally) - high because logic stay in the application
- dependencies - low (IDP needs to be integrated)
- onboarding - low (in case the resource need access control is already part of the domain model of authorization) or middle in case new roles need to be onboarded for the new resources
- access control complexity - simple lookup

**CISO**: *Although all teams can implement that solution fast and easy I have no real control that all teams are implementing the same logic based on our company rules. What also worries me is the dublicate effort on all teams to implement the same set of rules when handling similiar assets. But at least I can control and report centrally what roles are assigned to what users. In case we change rules in the company all teams need to change the rules locally as well - I can only hope that all apps have at least one active developer to implement the changes.*

- governance compliance enforcement - low (depends on all applications doing the right thing)
- governance compliance adaption - complex because all teams need to implement changes redundant - open for risks because of priotization, legacy or ignorance
- governance ruleset needs to be simple because otherwise we have too many roles to handle

## Attribute Based Access Control (ABAC)
Technically ABAC does something similiar like RBAC but does not need to assign a role activly. The decision for access is made on attributes of any thinkable characteristics of the user or in dependency of the resource or other circumstances as well as it is shown in the example below. 

```python
import jwt

# The resource that the user is trying to access
RESOURCE = "secret_document"

# The user's access token
TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJyb2xlIjoic3VwZXJfYWRtaW4iLCJjb21wYW55IjoiT3BlbkFpIiwiYXV0aG9yaXR5IjoicHVibGljIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"

# Decode the token to access the user's attributes
decoded_token = jwt.decode(TOKEN, verify=False)
user_role = decoded_token["role"]
user_company = decoded_token["company"]
user_authority = decoded_token["authority"]

# Check the user's attributes to see if they have access to the resource
if user_role == "super_admin" and user_company == "OpenAI" and user_authority == "private":
    print("Access granted to resource: " + RESOURCE)
else:
    print("Access denied to resource: " + RESOURCE)
``` 
Also other attributes of a user can be provided like time and location and can also be considered in real time. 

This exmaple is simple because the decision rule is done in the application itself locally. Usually ABAC does comes with more complex rules (policies) and a central management system to define and enforce them.

**Application Team**: *Similiar to RBAC this works for us as long as the rules are not getting to complicated and we also do not need to include external factors in realtime. Then the access control code will get messy in here. The defined ruleset on how to decide for the access is centrally documented but the rest is on us.*
- ownership (locally) - high because logic stay in the application
- dependencies - low (IDP needs to be integrated)
- onboarding - low because needed attributes are hopefully part of the token already
- access control complexity - simple lookup or risk of complex rules and too complex access control logic inside the app

**CISO**:  *The ruleset for deciding for each access can now be more specific for the use cases but I still rely on the decision making and enforcement on the app teams. User get now access without explicit assignment but automatically based on their characteristics.*

- governance compliance enforcement - low (depends on all applications doing the right thing)
- governance compliance adaption - complex because all teams need to implement changes redundant - open for risks because of priotization, legacy or ignorance
- governance ruleset is now able to project every needed usecase in more details without handling to many roles
- governance automation - user get access automatically based on their characteristics and not explicit assignments
- operations effort - No high availability management system for the rulesets is needed because only the documentation of rules is centrally orchestrated but not the enforcement which is needed in realtime as part of the IDP 

## Policy Based Access Control (PBAC)
A policy represents rules on how to decide if access to a resource should be allowed or not. These rules can be simple as in RBAC (user needs a role) or complex based on a lot of conditions and attributes. 

The biggest difference is the central management which is quite mandatory because applications might define access control rules locally but a policy makes sense to be defined centrally and also the decision making is here central the first time in our examples. 

```python
import jwt

# The resource that the user is trying to access
RESOURCE = "admin_panel"

# The user's JWT token
TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJhbGxvd2VkX3Jlc291cmNlcyI6WyJhZG1pbl9wYW5lbCJdfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"

# Decode the JWT token
try:
    payload = jwt.decode(TOKEN, SECRET_KEY, algorithms=["HS256"])
except jwt.ExpiredSignatureError:
    print("Token expired.")
except jwt.InvalidTokenError:
    print("Invalid token.")

# Check if the resource is in the list of allowed resources
if RESOURCE in payload["allowed_resources"]:
    # Give the user access to the resource
    print("Access granted to resource: " + RESOURCE)
else:
    # Deny the user access to the resource
    print("Access denied to resource: " + RESOURCE)
``` 

The last part now checks **not** the roles or attributes of the user but looks for the allowed resources provided in the token. The IdP makes the decision based on defined policies what resource the user shall access. These policies can be very simple and also rely on role assignements or complex based on multiple dimensions like attribute in real time. 

**Application Team**: *Now we are getting somewhere. The last ABAC rulesets were really complex and it took a great deal of us to keep them updated. This is now much simpler because the authorization is done in the IdP and we just add the lookup for the dedicated resources granted in the payload. Whenever the rules changes we don't need to change our app because the decision is made centrally. We can focus on the business logic.  
From the perspective of a product owner on the other side the onboarding of our app and the resources we handle are more complex. There is an application process to get the needed ruleset and resource onboarded and approved.*
- ownership (locally) - still high because the decision is made centrally but now the team can focus on the app domain only BUT the smaller and simpler the app the less this is an advantage
- dependencies - high because IdP and authorization decision needs to be handled externally and the resources need to aligned with the company domain model (if there is any) but the app can still accept similiar responses
- onboarding - middle for the ruleset onboarding for this app and resources BUT the smaller and simpler the app the less this is an advantage
- access control complexity - simple only because the decision making is done externally

**CISO**: *Finally this is what I dreamed of. The rulesets are defined centrally on our control and also the decision making for every single access request is done centrally. So also the reporting what resources are access by whom is easier that way. The auditors will love this.*

- governance compliance enforcement - high because decision making is central
- governance compliance adaption - easy because policies are managed centrally by all stakeholders and no app team needs to implement or change the rules locally 
- governance ruleset is able to project every needed usecase in more details without handling to many roles
- governance automation - user get access automatically based on their characteristics and not explicit assignments
- operations effort - high - High available management system for the decision making is needed 

### Orchestrated Authorization
The last example is IMHO one of the most central access control solutions one can think of. 

```python
import json
import requests

# The resource that the user is trying to access
RESOURCE = "admin_panel"

# The user's access token
TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJhbGxvd2VkX3Jlc291cmNlcyI6WyJhZG1pbl9wYW5lbCJdfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"

# Create a JSON payload with the resource and token
payload = json.dumps({"resource": RESOURCE, "token": TOKEN})
headers = {'Content-Type': 'application/json'}

# Send a request to the authorization server to validate the token and check access to the resource
response = requests.post("https://auth.example.com/validate", data=payload, headers=headers)

# Check the response from the authorization server
if response.status_code == 200:
    # The token is valid and the user has access to the resource
    print("Access granted to resource: " + RESOURCE)
else:
    # The token is invalid or the user does not have access to the resource
    print("Access denied to resource: " + RESOURCE)
```

This is close to the PBAC solution but sends the requested resource as part of the decision making to the authorization service. The authorization service does decide on the provided resource and the user if the access can be granted. The decision can be granted based on roles, attributes, other levels - sky is the limit. Important is in this approach that the decision making is totally delegated and the application just needs to enforce the access control based on the simple status_code. 

**Application Team**: *Works for us almost the same as the PBAC solution. Great is the simple access control logic we need to take care and we never need to change in our app. We can focus in full on the domain of our app and don't need to invest into the access control which is nice. On the other side we can not really control by what rules we grant access but the product owner will take care of this as part of the stakeholder. The onboarding is quite complex but luckily we are handling a lot of resources so this will pay off in the end. But imagine we are handling only one resource - this would be a hell of bottleneck for the onboarding but also on the application side. Doing access control locally based on one simple rule might be much faster in development and with less dependencies. 
We also experienced some performance issues with the IdP when issuing the access token so this payload handling might be an issue they need to take care of.*

- ownership (locally) - still high because the decision is made centrally but now the team can focus on the app domain only, BUT the smaller and simpler the app the less this is an advantage
- dependencies - very high because IdP and authorization decision needs to be handled externally and the resources need to aligned with the company domain model (if there is any) - if the resource is not aligned the authorization will respond with 401
- onboarding - middle for the ruleset onboarding for this app and resources, the smaller and simpler the app the less this is an advantage
- access control complexity - not existing because the app gets the yes or no decision for every request

**CISO**: *This brings tears into my eyes. 100% control about the decision making in real time and every time access is needed - best approach to reach Zero-Trust in this company. The applications do not have the effort of implementing any kind of access control logic except the bare minimum. This also helps in ithe changing nature of my organisation and the policies and rules we are defining. 
On the other hand I'm now responsible as the single point of failure in this company. Without this IAM solution no application can process requests and the provided payload for every request against IAM needs to be handled as efficient as possible. And I need to take care of the onboarding effort of integrations into the IAM solution.*

- governance compliance enforcement - high because decision making is central
- governance compliance adaption - easy because policies are managed centrally by all stakeholders and no app team needs to implement or change the rules locally 
- governance ruleset is able to project every needed usecase in more details without handling to many roles
- governance automation - user get access automatically based on their characteristics and not explicit assignments, but can also be based on roles
- operations effort - very high - High available management system for the decision making is needed and much performance is needed to handle the payloads of every request to decide for the specific resources

### Note
As already dropped in the disclaimer - ABAC, PBAC or even RBAC can be implemented locally or in central services centrally with Policy Enforcement Points (PEP) and Policy Decision Points (PDP) (https://csrc.nist.gov/publications/detail/sp/800-162/final). The point above were showing examples and does not deliver a full coverage of all possible implementations of such authorization models. 

# Summary of criterias on an auth model 
When deciding on the authorization "framework" for the company a couple of things need to be generally taken into account
- culture - what kind of services do you have and by what methods do your teams usually work
- how much risk can be accepted to also meet the needs of the app teams
    - banking for example gives not much room in "negotiation" of authorization model and the enforcement

## IAM acceptance criterias from DevOps / Agile perspective

- ownership of the teams - developing and decision making locally
- less dependencies to other services
- domain focus on the app
- onboarding and process efforts need to be non-existent (at best)

## IAM acceptance criterias from Security, Risk & Compliance

- governance compliance enforcement 
- governance compliance adaption 
- governance ruleset coverage
- governance automation
- operational efforts 

# Challenges for central authorization services
Especially in a microservice environment a complex and more centrally authorization logic could be to overwhelming for the apps with a clear domain context to be onboarded and handled. At the same time especially when resources are handled in such distributed small services a strong enforcing IAM solution makes sense from the security perspective. 

- central solutions are mostly an organisational nightmare for application owner because of the onboarding and continous management overkill 
    - mitigation: keep the onboarding simple if possible or open up to different processes dependent on complexity of the app and the handled assets
    - mitigation: self service, no approval flows for onboardings, keep the ownership at the application as long as the resource is handled locally

- officially companies are interested in eliminating shadow IT but these are mostly the high efficient services in delivering values

- modern cultures working with DevOps and Agile principles do wanna reduce dependencies and a central IAM is always a dependency with a lot of weight it comes with
    - mitigation: do not enforce complexity to applications if not needed otherwise apps will find ways around the rules and expose the company to maybe even more harm
    - differentiate use cases in the authorization model 
        - e.g. in software delivery SOD is generally dead because "you develop it - you own it" - here the authorization should be based on other control gates like peer reviews and attestations

# Migration of Applications
A delicate topic when it comes to Authorization is the migration of existing applications. 

- communication - you need a strategy to upfront involve the product owners but always keep in mind - your authorization does not really add any value to the app 
    - for the acceptance the teams need to be included in the definition of the ruleset, processes and such to get them onboard as soon as possible

- keep onboarding and management as simple as possible
    - offer self service for everything and no approval commitee - they need it they get it, no waiting time
    - worst case could be that an app needs to pay for this new authorization - this might kill all acceptance in the beginning 

- have an open mind in what apps and assets really need to be covered into the central authorization logic and which can stay stand alone
    - there is not one size fits all solution - even in IT security
    - the bigger the company the more diverse gets the IT landscape and their different needs

Consider all app teams as self thinking organisations with own objectives and ideas

# Conclusion
the more enforcement of authorization is provided centrally the more simple gets the technical integration for the teams but the higher is the dependency for them because the local domain context needs to aligned with the company. 

Sources:
- https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-162.pdf
- ChatGPT was a great assitant for the coding examples