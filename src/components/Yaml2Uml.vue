
<template>
  <div class="yaml2uml">
    <div class="file">
      <div v-if="!yaml">
        <h2>Select a yaml</h2>
        <input type="file" @change="onFileChange">
      </div>
      <div v-else>
        <button @click="removeFile">Remove file</button>
      </div>
    </div>
    <div class="picker">
      <h3>Class header color:</h3>
      <compact-picker v-model="classColor" />
    </div>
  </div>
</template>

<script>
import * as jsyaml from 'js-yaml'
import compact from 'vue-color/src/components/Compact.vue';

export default {
  name: 'Yaml2Uml',
  components: {
    'compact-picker': compact
  },
  data: function() {
    return {
      yaml: '',
      isOpenApi3: false,
      baseXml: `<?xml version="1.0" encoding="UTF-8"?><mxGraphModel dx="950" dy="598" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="850" pageHeight="1100" background="#ffffff" math="0" shadow="0"><root><mxCell id="0"/><mxCell id="1" parent="0"/></root></mxGraphModel>`,
      currentID: 1,
      xmlDoc: null,
      root :null,
      definitions: null,
      classColor: {hex: '#DAE8FC'}
    }
  },
  methods: {
    /**
     * Retrieve the file
     */
    onFileChange(e) {
      let files = e.target.files || e.dataTransfer.files;
      if (!files.length)
        return;
      this.parseFile(files[0]);
    },
    /**
     * Save the file to this.yaml
     * Send it for testing
     */
    parseFile(file) {
      let reader = new FileReader();
      reader.onload = (e) => {
        this.yaml = e.target.result;
        this.testYaml(this.yaml);
      };
      reader.readAsText(file);
    },
    /**
     * Clears this.yaml
     */
    removeFile: function (e) {
      this.yaml = '';
    },
    /**
     * Parses yaml to json and 
     * Ready the file for parsing, check yaml version
     * Prepare the base XMLDoc and sends definitions/schemas to build f()
     */
    testYaml: function(contents) {
      let jsonYaml = jsyaml.load(contents);
      this.isOpenApi3 = jsonYaml.openapi && jsonYaml.openapi.startsWith('3') ? true : false;
      let definitionSize =
        this.isOpenApi3 ?
          Object.keys(jsonYaml.components.schemas).length :
          Object.keys(jsonYaml.definitions).length;
      if (definitionSize < 1) {
        console.log('No model to load!');
        return;
      }
      console.log(`Loading ${definitionSize} models...`);
      let parser = new DOMParser();
      this.xmlDoc = parser.parseFromString(this.baseXml, "application/xml");
      this.root = this.xmlDoc.getElementsByTagName('root')[0];
      this.definitions = this.isOpenApi3 ? jsonYaml.components.schemas : jsonYaml.definitions;
      this.buildXml();
    },
    /**
     * For each definition, build and XML mxCell element
     * Sends the complete XML a new window
     */
    buildXml: function() {
      let classPosition = 1;
      for (let definition in this.definitions) {
        let newClass = this.buildClass(definition, classPosition);
        classPosition++;
      }
      this.displayXml();
    },
    /**
     * Set up a mxCell element of type swimlane and its mxGeometry counterpart
     Then for each property of the definition, build the corresponding mxCell
     */
    buildClass: function(definition, position) {
      let style = `swimlane;fontStyle=0;childLayout=stackLayout;horizontal=1;startSize=26;fillColor=${this.classColor.hex};horizontalStack=0;resizeParent=1;resizeParentMax=0;resizeLast=0;collapsible=1;marginBottom=0;swimlaneFillColor=#ffffff;`;
      let width = 140;
      let spacing = 20;
      let y = 10;
      let fieldNumber = 0;
      let props = {};
      if (this.definitions[definition].properties) {
        props = this.definitions[definition].properties;
        fieldNumber = Object.keys(props).length;
      } else if (this.definitions[definition].type == 'array') {
        props = this.definitions[definition].items;
        fieldNumber = 1;
      } else if (this.definitions[definition].allOf) {
        props = this.manageAllOf(this.definitions[definition].allOf);
        fieldNumber = Object.keys(props).length;
      }
      let height = 26 + fieldNumber * 26;
      let newClass = this.xmlDoc.createElement('mxCell');
      newClass.setAttribute('id', ++this.currentID);
      newClass.setAttribute('value', definition);
      newClass.setAttribute('style', style);
      newClass.setAttribute('vertex', 1);
      newClass.setAttribute('parent', 1);
      let newGeometry = this.xmlDoc.createElement('mxGeometry');
      if (position % 4 == 0) {
        y = position * height + spacing;
      }
      newGeometry.setAttribute('x', position * width + spacing);
      newGeometry.setAttribute('y', y);
      newGeometry.setAttribute('width', width);
      newGeometry.setAttribute('height', height);
      newGeometry.setAttribute('as', 'geometry');
      newClass.appendChild(newGeometry);
      this.root.appendChild(newClass);
      let iterator = 1;
      let parent = Object.assign(this.currentID);
      if (this.definitions[definition].properties || this.definitions[definition].allOf) {
        for (let property in props) {
          let type = props[property].type;
          this.buildField(property, type, iterator, parent);
          iterator++;
        }
      } else {
        // #ref/components/schema/Pet returns [Pet]
        let property = /[^\/]+$/.exec(props.$ref);
        this.buildField(`[${property}]`, 'array', 1, parent);
      }
      return newClass;
    },
    /**
     * Create an mxCell and mxGeometry for a particular property
     */
    buildField: function(property, type, iterator, parent) {
      let style = 'text;strokeColor=none;fillColor=none;align=left;verticalAlign=top;spacingLeft=4;spacingRight=4;overflow=hidden;rotatable=0;points=[[0,0.5],[1,0.5]];portConstraint=eastwest;';
      let newField = this.xmlDoc.createElement('mxCell');
      newField.setAttribute('id', ++this.currentID);
      newField.setAttribute('value', `${property} : ${type}`);
      newField.setAttribute('style', style);
      newField.setAttribute('vertex', 1);
      newField.setAttribute('parent', parent);
      let newGeometry = this.xmlDoc.createElement('mxGeometry');
      newGeometry.setAttribute('y', 26 * iterator);
      newGeometry.setAttribute('width', 140);
      newGeometry.setAttribute('height', 26);
      newGeometry.setAttribute('as', 'geometry');
      newField.appendChild(newGeometry);
      this.root.appendChild(newField);
      return newField;
    },
    /**
     * Returns an object merged allOf ref
     */
    manageAllOf: function(allOf) {
      let obj = {};
      let ref = '';
      // forced to make 2 loops
      // each allOf can have either type: object + $ref
      // or properties + $ref
      // properties and $ref are not necessary in the same item
      for (let item in allOf) {
        if (allOf[item].$ref) {
          ref = /[^\/]+$/.exec(allOf[item].$ref)[0];
        }
      }
      for (let item in allOf) {
        if (allOf[item].properties) {
          allOf[item].properties[ref] = new Object({type: 'allOf'});
          obj = allOf[item].properties;
        }
      }
      return obj;
    },
    /**
     * Serialize XML file and propose a download
     */
    displayXml: function() {
      let s = new XMLSerializer();
      let blob = new Blob([s.serializeToString(this.xmlDoc)], {type: 'text/xml'});
      let filename = 'yaml.xml';
      if(window.navigator.msSaveOrOpenBlob) {
          window.navigator.msSaveBlob(blob, filename);
      }
      else{
          var elem = window.document.createElement('a');
          elem.href = window.URL.createObjectURL(blob);
          elem.download = filename;        
          document.body.appendChild(elem);
          elem.click();        
          document.body.removeChild(elem);
      }
    }
  }
}
</script>

<style scoped>
  span {
    padding-top: 30px;
  }
  .picker {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
</style>
