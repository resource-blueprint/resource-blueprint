# Gist API
Gist API.

## API Entry Point
[Gists@collection][]

## Resource Gists
The Gists resource is a list of individual Gist pastes.

### Attributes
The properties of a gist collection (Gists resource).

+ total_count (number) ... The total count of gists in the actual collection/selection.
    + Traits
        + profile_type: semantic

+ items (array) ... An array of embedded individual Gist entities.
    + Traits
        + profile_type: semantic
    + Embedded Entities
        + [Gist][]        

### Affordances
Link relations of a gist collection.

+ list ... List the complete, unfiltered collection.
    + Traits
        + profile_type: safe      

+ create ... Creates a gist, adds a gist to collection.
    + Traits
        + profile_type: unsafe

  + Attributes
        + description (string) ... Description of the gist.
            + Traits
                + profile_type: semantic

        + content (string) ... Content of the gist.
            + Traits
                + profile_type: semantic

+ search ... Filters results based on parameters.
    + Traits
        + profile_type: safe

    + Parameters
        + search_by (string) ... Keyword to search on.
            + Traits
                + profile_type: semantic

        + search_by_attribute (string) ... Attribute to apply search to.
            + Traits
                + profile_type: semantic

            + Values
                + description
                + content

+ first
+ previous
+ next
+ last

### States
States / state machine of a gist collection.

+ collection (entry point)
    + Affordances
        + self(list) -> collection
        + create -> [Gist@active][]
            + Conditions
                + can_create
                + can_update

        + search -> selection
        + next -> selection
        + last -> selection

+ selection
    + Affordances
        + self(search) -> selection
        + list -> collection
        + create -> [Gist@active][]
            + Conditions
                + can_create
                + can_update

        + first -> selection
        + previous -> selection
        + next -> selection
        + last -> selection

### HTTP
HTTP protocol-specific implementation. 

+ list: GET /gists
    + Response 200
        
        [Gists][]

+ create: POST /gists
    + Request (application/json)

            {
                "description": "Description of Gist",
                "content": "String Content"
            }

    + Response 201

        [Gist][]

+ search: GET /gists{?search_by,search_by_attribute}
    + Response 200

        [Gists][]

+ first: GET /gists?page=1{;per_page}
    + Response 200

        [Gists][]

+ previous: GET /gists{?page,per_page} 
    + Response 200

        [Gists][]       

+ next: GET /gists{?page,per_page}
    + Response 200

        [Gists][]

+ last: GET /gists{?page,per_page}
    + Response 200

        [Gists][]

### TCP
...

### COAP
...

### <other protocol>
...

### Media Types
+ application/hal+json

+ application/vnd.siren+json

+ application/json
    
    ```json
    {
        "id": "012345",
        "name": "My Gist",

        ...
    }
    ```

## Resource Gist

### Attributes
+ id
+ description
+ content
+ created_at
+ author
    + name

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
