# Minutes from the REST API Review Board 4/21/2020

These notes were captured during the meeting of the REST API Review Board on 4/21/2020.  They do not represent a complete conversation.

## Terminology

The first discussion was around terminology.  There are three levels of changes that need to be discussed:

1. **Compliance changes** - changes that may be breaking, but have to be done to an existing API due to security or compliance reasons.
2. **Breaking changes** - major changes to an API that will break an SDK user due to code generation.
3. **Evolutionary changes** - minor, additive changes to an API to support new features of the services.

## When to do a version bump

All changes (except compliance changes) must version bump.  We should be encouraging service teams to use preview versions and release GA versions of the API less frequently.  This allows the service to evolve without rapid and expanding versions of the API.

When using SDKs, we want customers to not break as they move forward.  Take, for example, a customer that wants to take advantage of a small additive feature, but moving to the newer version of the API results in breaking changes - fallout that they never considered previously.

## Code gen

There was a lot of discussion on the effects of code generation and how the SDK may break even though the API didn't technically have a breaking change.  This conversation was tabled - our job is to get the APIs right; Metadata changes (in the Swagger definition) have the potential to cause breaking changes without a similar breaking change in the API.

Participants noted that tooling is available for detecting breaking changes on the codegen when working in azure-rest-api-specs repo.  Perhaps we can productize this?

## How to reason about changes?

Jeffrey provided a table that allows service teams to reason about the changes to properties in an API:

| Change | Input | Output |
|========|=======|========|
| Remove | Major | Major  |
| Add optional | Minor | Major |
| Add required | Major | Major |
| Data type change | Major | Major |
| Format change | Major | Major |
| Int widens | Minor | Major |
| Int narrows | Major | Minor |
| Add enum | Minor | Major |
| Remove enum | Major | Major |
| Optional to required | Major | (See add) |
| Required to optional | Minor | Major |

## Further work

We need four further meetings (at least) to close on this topic:

1. SDK / Codegen processes
2. Enforcement
3. API Evolution processes for service teams
4. API versioning for additive changes

## Chat log

The following is the chat log (raw)

[9:13 AM] Darrel Miller
    In Azure API Management, we avoided using the term breaking change because it has too much baggage. We chose to use "Version" to indicate changes that a a client "opts-in" to. I.e. you change a version identifier in the URL and a "Revision" where changes are made by the server where the customer gets those changes automatically without doing anything.
​[9:18 AM] David Justice
    Gareth can you expand on PUT vs ?
​[9:19 AM] David Justice
    Are folks using PATCH instead?
​[9:19 AM] Anne Loomis Thompson
    This was my suggested rewrite: [https://github.com/microsoft/api-guidelines/pull/192#discussion_r410769561](https://github.com/microsoft/api-guidelines/pull/192#discussion_r410769561)
​[9:19 AM] Gareth Jones
    yes
(1 liked)​
[9:19 AM] Darrel Miller
    Most Graph API use PATCH
​[9:19 AM] Gareth Jones
    exactly that
​[9:19 AM] Gareth Jones
    From what I've seen PATCH is more dominant broadly too
​[9:20 AM] Johan Stenberg
    [https://github.com/johanste/serviceapievolution/blob/503667bb8047b3b0e36765e9769c44a0f2756ae7/guidance.md](https://github.com/johanste/serviceapievolution/blob/503667bb8047b3b0e36765e9769c44a0f2756ae7/guidance.md)
​[9:29 AM] Ryan Sweet
    Devil's advocate discussion fulcrum: what would be the most customer friendly stance (absent consideration of the burden on the development teams)?
[9:30 AM] Ryan Sweet
    Hypothesis: the majority of customers would prefer to see that any changes to the service or metadata result in a version bump
[9:31 AM] Steve Faulkner
    FWIW we frequently do additive changes in Cosmos REST APIs with no version bump. I don't think I've ever seen a customer complain about that.
(2 liked)
[9:33 AM] Jeffrey Richter
    **Insert table provided earlier**
[9:43 AM] David Justice
    Should we encourage services to introduce features more often in preview versions and less often mint "GA" versions?
(1 liked)​
[9:43 AM] Steve Faulkner
    Jeffrey Richter Would also like to see a row/meeting/examples for behavior changes (ie changes only detectable at runtime)
​[9:44 AM] Mark Cowlishaw
    Jeffrey Richter Does minor map to 'forward compatible' changes, where major is essentially non-forward-compatible changes?
​[9:45 AM] Darrel Miller
    The Microsoft REST guidelines actually say that major.minor is how services should be versioned [https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#121-versioning-formats](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#121-versioning-formats) The Date based format was intended to group multiple services.
​[9:45 AM] Johan Stenberg
    ..and the implied assumption is also that major versions are extremely rare...
[9:45 AM] Gareth Jones
    Ryan Sweet  hard disagree with your hypothesis.  From our experience most customer s would prefer that the majority of change would happen with no version change whatever.
(1 liked)​
[9:46 AM] David Justice
    Seems we'd also do well to deliver features in preview, stacking the changes and iterating on those apis.
​[9:47 AM] David Justice
    optional changes, preview stacking and PATCH usage == (laugh)
​[9:47 AM] Jeffrey Richter
    My thinking has changed on this recently. I think MINOR should map to a change that doesn't break old api-versions. And MAJOR should map to a change that does break old api-versions. If we can all agree to this, then I think we should apply this thinking to my table. What's there might be right (or close) but I'm open to finer discussion on the cell values.
​[9:48 AM] Gaurav Bhatnagar
    optional properties should not require a new api-version. its not ideal, but i think we are in a state where we have to make that a norm and live with it. we need to tell the clients that they have to handle such change.
(1 liked)
[9:48 AM] Jeffrey Richter
    Service teams can do minor updates whenever they want (although we may not want every 2 weeks for other reason). Major updates requires a BIG review and discussion as to how to minimize the negative impact to existing customers.
​[9:48 AM] Gaurav Bhatnagar
    it slows down
​[9:48 AM] David Justice
    Gaurav, if that guidance is paired with the use of PATCH, it works
​[9:49 AM] Gaurav Bhatnagar
    yes agreed David
(1 liked)​
[9:49 AM] Gaurav Bhatnagar
    i would go one step further
​[9:50 AM] Jeffrey Richter
    Gaurav Bhatnagar: optional properties DO require a new api-version. Otherwise a client can't know if the optional property they want is guaranteed to be there.
​[9:50 AM] Anne Loomis Thompson
    My internet just cut out
​[9:52 AM] Gaurav Bhatnagar
    Jeffrey Richter  and I am arguing against making it require a new api-version. its not practical. it has been a rule since ARM existed but we are in this hole. there are too many challenges enforcing it. and if we cant enforce it, there is not point having that guidance. clients can ignore properties it doesnt understand.
(1 liked)
​[9:52 AM] Mark Cowlishaw
    One thing that definitely resonates is including specific, real examples of the changes that fit in each category, and why they constitute major or minor changes
(1 liked)​
[9:53 AM] David Justice
    we should record these
(1 liked)
[9:53 AM] Gaurav Bhatnagar
    and there is no way in Azure api-versioning scheme to know when something is minor or something is major change.
​[9:54 AM] Mark Cowlishaw
    correct, this stands in for forward-compatible or non-forward compatible changes, I think, and perhaps whether an api-version change is required
​[9:54 AM] David Justice
    These conversations would do well to be preserved to help the people who will inevitably be discussing this 5 years from now.
Edited
​[9:54 AM] Mark Cowlishaw
    although for now, we are saying api-version changes for everything
​[9:55 AM] Gaurav Bhatnagar
    Mark - i would think hard about doing it
​[9:55 AM] Mark Cowlishaw
    I am not saying I agree, I am saying that is the current stance
​[9:57 AM] Jeffrey Richter
    Gaurav Bhatnagar: I see what you want and see why but I'd prefer if there was a way to enforce it. For SDKs, the optional field/property has to be added to a type in the SDK. How can customer code know whether it can expect the value to be a good value or now? Nullable is possible in some languages but is not idiomatic in other languages. Also, when you say option properties, you need to be clear about optionally being sent TO the service or FROM the service.  SDKs might be able to tolerate better the optional being sent TO the service but hard to tolerate the optional being sent FROM the service.
​[9:59 AM] Steve Faulkner
    The "api-version changes for everything" is worth at least having a meting about. I can only speak for Cosmos, but sounds like we aren't the only service with issues. Even if it is the current guideline.
​[9:59 AM] Johan Stenberg
    On a side note, how will a service behave if the added property does not yet exist on the cloud I'm calling? Reject the call? For example, consider the scenario of adding a property/capability to a service API version that already has been made available on Azure Stack (or in a containerized form) - there is no guarantee that the Azure Stack instance has been updated to support the new property in that case.
(1 liked)
[10:01 AM] Gaurav Bhatnagar
    Jeffrey - agree with you on the usecase you mention. Is it an unsolvable problem on the client side? I would like us to explore if there any other options here? How do other platforms solve it? GCP API version guidelines indicate such changes to be safe - [https://cloud.google.com/apis/design/compatibility](https://cloud.google.com/apis/design/compatibility).
​[10:03 AM] Khaled Henidak (KAL)
    Context really matter: Adding an optional field might look safe but it isn't
​[10:04 AM] Khaled Henidak (KAL)
    consider v1 Person{firstName: string} and v2 `Person{firstName:string, lastName: string}`
​[10:04 AM] Khaled Henidak (KAL)
    Now client 1(rev'ed) puts a person {firstname:kal, lastname:microsoft}
​[10:04 AM] Khaled Henidak (KAL)
    client2 (not rev'ed) gets Kal and modifies firstname {firstName: Khaled} then puts the data
​[10:05 AM] Khaled Henidak (KAL)
    what you will end up is unintented data loss
​[10:05 AM] Khaled Henidak (KAL)
    because the client replaced the entire payload with what they think is right
​[10:05 AM] Khaled Henidak (KAL)
    which is not
​[10:06 AM] Khaled Henidak (KAL)
    You may say let us treat put as patch. i.e. if the client didn't specifiy last name then we will not update it..
​[10:06 AM] Khaled Henidak (KAL)
    But what if the client intent to actually remove LastName value?
[10:06 AM] Mark Cowlishaw
    Basically, allowing an wpi-version to change means that api-version is essentially meaningless for regions / containerized/airgapped clouds
​[10:06 AM] Darrel Miller
    This is why AtomPub has this clause [https://bitworking.org/projects/atom/rfc5023#rfc.section.9.3](https://bitworking.org/projects/atom/rfc5023#rfc.section.9.3)  OData adopted this also.
​[10:07 AM] Khaled Henidak (KAL)
    that is a protocol spec. we can force it on our sdks, but can we force it on other clients?
​[10:07 AM] Mark Cowlishaw
    that is allowing a change without updating the api-version
​[10:07 AM] Khaled Henidak (KAL)
    maybe we should have it as a rule for any azure client
​[10:08 AM] Khaled Henidak (KAL)
    That becomes even an issue with create or replace methods that we offer in our sdks.
​[10:08 AM] Khaled Henidak (KAL)
    because the intention becomes even less clear
[10:09 AM] David Justice
    What if versioning was automated? Say, a service team iterates on vNext in preview, staging the changes they want. At some interval, a new version is cut. A diff of that version occurs and the new version is calculated based on the impact of the changes in the API surface.

    The machinery needed for versioning is a part of the system, not overhead of the service team. Versioning does not slow the team. All versioning is enforced and done uniformly.

Edited​
[10:09 AM] Johan Stenberg
    I think we need to have a separate discussion on the use of PUT for ARM.
(2 liked)​[10:12 AM] Johan Stenberg
    I don't see how a service team can avoid thinking about versioning. There are many things that require the v1 of an API to be designed in a way that will not cause future friction (e.g. should this number be a 32 bit value or a 64 bit value, is a boolean a good type to use etc.). A machinery that only detects versioning issues once they have two versions to compare will leave a lot of people painted into corners.
​[10:12 AM] Johan Stenberg
    That doesn't mean that tooling cannot help - but we certainly need versioning guidance for the first version of an API as well - including linters etc.
​[10:12 AM] David Justice
    still need to design the api
​[10:13 AM] David Justice
    I don't want to take jobs away for doing important things like make a api usable and well thought out.
​[10:13 AM] Khaled Henidak (KAL)
    I don't think the issue is just tooling. The issue goes to building know how and maybe provide scafolding.
​[10:14 AM] David Justice
    There is friction asking teams to version their services for every little change. That needs to be solved to increase feature velocity.
[10:14 AM] Johan Stenberg
    Does it?
​[10:14 AM] Johan Stenberg
    For GA versions of API versions?
​[10:15 AM] David Justice
    Seems that if automating a process with a heavy burden does not make it easier, then we are doing something wrong.
Edited
​[10:15 AM] Johan Stenberg
    How heavy is the burden?
​[10:15 AM] Khaled Henidak (KAL)
    I think we are discussion this without the cost. e.g. the cost of doing this and breaking your client and getting CRI storm of extremely difficult buts to debug vs the cost of investing a little into versioning. That btw goes to feature velocity and jedi
​[10:16 AM] Gaurav Bhatnagar
    In my mind, there are 3 challenges we need to address as it relates to api-versioning guidance 1. Enforcing (specially for things like adding optional properties, optional read only properties in the response, adding values to enum ) 2. Explosion of API-versions  - guidance and enforcing 3. Overhead of adding new api-version for every small change on the service side
​[10:18 AM] Khaled Henidak (KAL)
    i think optional properties are an overloaded term in Azure. I have looked at a couple of CRP versions they are riddled with optional properties but i know for a fact that not providing most of these properties will yeild into validation errors
​[10:19 AM] Gaurav Bhatnagar
    actually there is a 4th one -  4. How does customer know when a new api-version contains a minor additive change or when it contains a major change
​[10:19 AM] Mark Cowlishaw
    Can we use preview versions, or a similar concept, LTS versions to solve the problem for ARM - that is, allow changes in non-LTS versions and have LTS versions with strict guidelines be the only ones ported to other clouds
(1 liked)​
[10:19 AM] Gaurav Bhatnagar
    Khaled - wow
[10:19 AM] Mark Cowlishaw
    or would that just force all the changes into non-LTS versions?
​[10:20 AM] Khaled Henidak (KAL)
    yup. you know that network profile is marked as optional. I know for a fact that you can not create a vm without network and storage profiles
​[10:20 AM] Johan Stenberg
    It would be interesting to me to (re)explore the concept of frozen vs. non-frozen APIs.

    * There is a "recommended" version of an API. This is guaranteed to not change.
    * There is a "latest" (but still supported/has SLA etc.) version of the service version where new fields may be added.
    * An API is automatically frozen (by necessity) as soon as it is available on multiple "clouds" where deployment happens at potentially different schedules.

(2 liked)​
[10:20 AM] Khaled Henidak (KAL)
    so there is that as well
​[10:20 AM] Mark Cowlishaw
    Khaled Henidak (KAL) Yes, that is common across RPs, as we start to create toollking for more services
​[10:21 AM] Johan Stenberg
    > yup. you know that network profile is marked as optional. I know for a fact that you can not create a vm without network and storage profiles

    ...however, you can PUT to an existing VM without providing those properties... 
[10:21 AM] Khaled Henidak (KAL)
    Johan Stenberg my friend. Optional is optional.
​[10:22 AM] Khaled Henidak (KAL)
    you are telling me mr user i don't need these things from you. I can default it for you. that means everytime you write you don't need to provide them
​[10:22 AM] Johan Stenberg
    Khaled Henidak (KAL), I wasn't the designer of those APIs! (smile)
​[10:22 AM] Khaled Henidak (KAL)
    the concept of optional on update only does not exist
​[10:22 AM] Khaled Henidak (KAL)
    LOL
