![logo](https://raw.github.com/apiaryio/api-blueprint/master/assets/logo_apiblueprint.png)

&nbsp;

> “... there’s nothing about URLs, nothing about status codes. It’s all about state machines and media types.”
>
> *Steve Klabnik, “Designing Hypermedia APIs.”*

# Resource Blueprint
Proposal of resource-oriented, protocol-independent [API Blueprint](http://apiblueprint.org). Focused on 
modeling API resources, their attributes, [affordances](http://en.wikipedia.org/wiki/Affordance) and an API state machine.

## Concepts
The following highlights the API design concepts underlying a Resource Blueprint.

* **Semantically define data and affordances** (link relations).
* **Define state-machines** representing a resource.
    * Includes transitions in different states (business rules) and the conditions (permissions) for their inclusion 
    in a response.
    * Ideally, this should ultimately draw the state-machine as part of the design process.
    * Will be used mock the API to act as a true hypermedia API based on state.
* **Specify supported media-types.**
    * The API Blueprint parser will generate sample representations based on the semantics, state-machine and 
    known media-types.
* **Specify the entry point to the resource.**
* **Adding metadata to elements** that could be used to define a profile (e.g. [ALPS](http://alps.io/spec/index.html)) 
from the blueprint and other uses in the machine-readable output of the API Blueprint parser.
* **Enter protocol specific information** as an implementation detail well after having designed the actual API.
    * Initially only supports HTTP protocol.
    * The only actual URI you will see in the human-readable version is the entry point, even though you can enter 
    URIs in the blueprint.

APIs associated with specific resources can be designed individually as parts of an overall API that may span several
resource blueprints to represent the overall API application state-machine.

---

## Quick Links
+ [Use Case - Gist API State Machine](#def-use-case-api)
+ [Full Gist API Example](#def-full-example)
+ [Resource Blueprint Syntax](#def-syntax)
+ [Concepts in Detail](#def-concepts-detail)
+ [Credits](#def-credits)

---

<a name="def-use-case-api"></a>
## Use Case - Gist API State Machine
The example API used throughout this proposal is of an imaginary GitHub's Gist-like service. This example API demonstrates **Gist API State Machine** on the `Gists` (collection) and `Gist` (entity) resources.

### Gists Resource 
An entry point to Resource Blueprint Gist API is the `Gists` resource in its `collection` state. This resource has two states: `collection` and 
`selection`:

![fig1](assets/Gist%20State%20Machine%20001.png)

This diagram translates to the following resource blueprint:

```Markdown
## Resource Gists

### Attributes
+ total_count
+ items

### Affordances
+ list
+ create
+ search

### States
+ collection (entry point)
    + Affordances
        + self(list) -> collection
        + create -> [Gist@active][]
        + search -> selection

+ selection
    + Affordances
        + self(search) -> selection
        + list -> collection
        + create -> [Gist@active][]
```

> **Note:** Both states also serve as factories for individual `Gist` entities.


### Gist Resource
A `Gist` resource created by the `Gists`'s `create` affordance has following two states: `active` and `archived`:

![fig2](assets/Gist%20State%20Machine%20002.png)

Represented in resource blueprint as:

```
## Resource Gist

### Attributes
+ id
+ description
+ content

 ...

### Affordances
+ show 
+ edit
+ delete
+ archive
+ restore
+ author

### States
+ active
    + Affordances
        + self(show) -> active
        + edit -> active
        + delete -> (exit point)
        + archive -> archived
        + author -> [Author@default][]

+ archived
    + Affordances
        + self(show) -> archived
        + restore -> active
        + author -> [Author@default][]
```

### Embedded Entities
Finally, an individual `Gist` resource embedded in the `items` attribute of a `Gists` resource can be accessed directly via its `self` affordance:

![fig3](assets/Gist%20State%20Machine%20003.png)

```
## Resource Gists

### Attributes
+ total_count (number) ... The total count of gists in the actual collection/selection.

+ items (array) ... An array of embedded individual Gist entities.
    + Embedded Entities
        + [Gist][]

### Affordances
+ list
+ create
+ search

 ...
```

<a name="def-full-example"></a>
## Full Gist API Example
The complete, GitHub-rendered [Use Case API](#def-use-case-api) can be found in the [Gist API.md](examples/Gist%20API.md) file. The complete example discusses `Gists collection` resource in full detail and and `Gist entity` resource in minimal detail.

View its [raw version](https://raw.github.com/apiaryio/api-blueprint/resource-blueprint/examples/Gist%20API.md) to get its source blueprint. Refer to [Resource Blueprint Syntax](#def-syntax) for explanation syntax constructs.

<a name="def-syntax"></a>
## Resource Blueprint Syntax
A Resource API Blueprint follows [regular markdown](http://daringfireball.net/projects/markdown/syntax) syntax rules with semantic meaning of some specific construct and/or keywords.

### Keywords
New keywords introduced in the Resource Blueprint are:

+   `API Entry Point` ... Denotes state machine entry point referencing a resource and its state.

+   `Resource` ... Definition of a resource `Resource <resource name>`.

+   `Attributes` ... Definition of resource attributes. *Alt keyword:* `Properties`.


+   `Embedded Entities` ... Reference to one or more resource entities embedded in given resource attribute.

+   `Affordances` ... Definition of **all** resource's affordances. *Alt keywords:* `Transitions`, `Actions` or `Link Relations`.
        
    *Optional:* A resource definition without any affordance should assume one default "show/self" affordance.

+   `States` ... definition of resource states and state transitions invoked using affordances.
    Each state should list all available affordances and the associated final state using the relevant affordance in the `<affordance> -> <new state>` format.

    The new state might be a reference to another resource state (see Referencing Syntax bellow). One affordance per state should be marked as `self` to represent the default "retrieve" affordance. 

    *Optional:* Defining a state radically improves experience and possibilities in the following Resource Blueprint tool chain. However it should be valid to define a resource without states definition (and therefore without the `API Entry Point`) albeit it practically means that the given resource' state cannot be changed. 

+   `Conditions` ... list of conditions for an affordance to be available in the respective state.
    Associated with **business rules** tied to an affordance being available or present in a response. 
    *Alt keywords:* `Permissions` or `Rights`

    *Optional:* Conditions on affordances are completely optional. Absent conditions, an affordance should be considered always present in a response.

+   `HTTP`, `TCP`, `COAP`, `<other protocol>` ... Protocol specific implementation of respective affordances.

    *Optional:* Protocol section should be optional. If not present, in the case of HTTP, a default method (GET?) should be expected with a default URI constructed from the resource name and possibly affordance parameters and/or name. To be decided. 

+   `Media Types` ... List of supported media type representations of respective resource attributes and affordances.
    Might include an explicit example asset.

    *Optional:* If media types are not present a default media type should be provided. Consider using `application/hal+json` and `application/vnd.siren+json`.

    If a media type is specified but no explicit example is provided, a default representation using attributes and affordances should be
    generated for known media-types. For not recognized media-types an explicit example should be included.

+   `Traits` ... An arbitrary API Blueprint object traits. Used to specficy additional object characteristic or metadata related to object. Refer to [API Blueprint Traits](https://github.com/apiaryio/api-blueprint/issues/47) for details.
    
    > **Note:** The Gists Resource of the full Gist API example demonstrates additional metadata to be used for generating an [ALPS profile](http://alps.io/spec). 

> **Note:** Consider using `Alt keywords` instead of proposed keywords to improve clarity for general audience.

### Naming Attributes and Affordances
Where possible use undecorated plain text. In the case of a clash with keyword and/or the need for escaping (to not collide with Markdown syntax) use [Markdown code span element syntax](http://daringfireball.net/projects/markdown/syntax#code) e.g. `` `Attributes` ``.

### Referencing Syntax
+   Reference to a resource representation: `[<resource>][]`
+   Reference to a resource state: `[<resource>@<state>][]`
+   Reference to a resource affordance: `[<resource>.<affordance>][]`

<a name="def-concepts-detail"></a>
## Concepts in Detail 
### Noun vs. Verb Affordances (link relations)
The concepts of "application state" and "resource state" are a useful level abstraction as one thinks about an API and 
the affordances that exist in a response. From an API standpoint it is all transparent, you just get data and 
links/forms. But some affordances will affect/cycle resource state more like "actions" while other affordances will 
link to other resources and change "application state" more like nouns. 

The noun driven idea about relations is strongly tied to the idea of polling options to find uniform interface verbs w/r 
to a particular resource and moving forward. This perspective is particularly limiting in an M2M 
situation where you want a machine to semantically understand a relation vs. a verb in the uniform interface and very possibly 
have a situation where a service can specify the verb as part of the metadata associated with a link in a response. 

Thus, a client need only understand the semantic significance of the link relation and move forward. Verb related 
transitions/relations would be completely appropriate in that case (and more semantically rich than nouns). 
Accommodating this type of idea is important vs. the more human-documentation centric position of understanding 
what the verbs in the uniform interface do as a sole approach.

### Resource States
Resource state machine affordances are two-fold. There are affordances that change state in such a way that the 
set of _possible_ affordances change and those that exercise data-state wherein some property(ies) of the resource are 
modified but the _possible affordances_ are not affected by those changes. There are infinite of the latter. However,
based on business rules, there are a finite number of unique sets of _possible_ affordances. These sets of affordances
define the actual resource states.

### Context and Conditions
When a resource is in some particular state, the only affordances at any time that may be returned are a fixed set. 
Whether or not an affordance is returned in a response is a function of context and conditions. These do not represent 
a resource state in the state-machine, but rather a filter on the available affordances that the context gives access 
to. 

E.g. if the state has an `edit` affordance and you don't have rights to edit it, you don't see the affordance. This 
is not a different state of the resource from a state-machine standpoint of the resource, but rather of what you can 
do with it at some point in time for the given context and conditions (permissions). To the client, it may well look 
like a different state, but from the internals of an API it is not a separate state of a resource.

<a name="def-credits"></a>
## Credits
+ **Authors:** Mark W. Foster <[@fosdev](https://github.com/fosdev)>, Zdeněk Němec <[@zdne](https://github.com/zdne)>
+ **Created:** 2013-10-30
+ **Updated:** 2014-01-24

## License
MIT License. See the [LICENSE](LICENSE) file.

