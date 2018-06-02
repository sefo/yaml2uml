# yaml2uml
Convert [Swagger](https://editor.swagger.io/) YAML files (*definitions* part) to an XML file that can be opened on https://www.draw.io/

Head over to https://sefo.github.io/yaml2uml/ and try it out.

Supports swagger 2.0 and OpenApi 3 models (definitions *vs* components.schemas)

Supports array refs

```
  TestArray:
    type: "array"
    items:
      $ref: "#/definitions/Tag"
```

Supports allOf

```
  Dog:
    allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        properties:
          bark:
            type: boolean
          breed:
            type: string
            enum: [Dingo, Husky, Retriever, Shepherd]
```

Current TODOS:

- Fix position problem
  - Class diagrams are not aligned correctly on the grid at the moment.
- Download XML
  - For the moment the XML is only visible in the console
- Add property types
  - Properties are not typed. Later parse properties.type
- Add colors
  - Allow changing cell style
- Add Links between classes
  - Parsing category: $ref: "#/definitions/Category"
