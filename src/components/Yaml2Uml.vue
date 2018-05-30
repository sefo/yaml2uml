
<template>
  <div class="yaml2uml">
    <div v-if="!yaml">
      <h2>Select a yaml</h2>
      <input type="file" @change="onFileChange">
    </div>
    <div v-else>
      <button @click="removeFile">Remove file</button>
    </div>
  </div>
</template>

<script>
import * as jsyaml from 'js-yaml'

export default {
  name: 'Yaml2Uml',
  data: function() {
    return {
      yaml: '',
      baseXml: `<?xml version="1.0" encoding="UTF-8"?><mxGraphModel dx="950" dy="598" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="850" pageHeight="1100" background="#ffffff" math="0" shadow="0"><root><mxCell id="0"/><mxCell id="1" parent="0"/></root></mxGraphModel>`,
      currentID: 1,
      xmlDoc: null,
      root :null,
      definitions: null
    }
  },
  methods: {
    onFileChange(e) {
      let files = e.target.files || e.dataTransfer.files;
      if (!files.length)
        return;
      this.parseFile(files[0]);
    },
    parseFile(file) {
      let reader = new FileReader();
      reader.onload = (e) => {
        this.yaml = e.target.result;
        this.testYaml(this.yaml);
      };
      reader.readAsText(file);
    },
    removeFile: function (e) {
      this.yaml = '';
    },
    testYaml: function(contents) {
      let jsonYaml = jsyaml.load(contents);
      let definitionSize = Object.keys(jsonYaml.definitions).length
      if (definitionSize < 1) {
        console.log('No definition to load!');
        return;
      }
      console.log(`Loading ${definitionSize} models...`);
      let parser = new DOMParser();
      this.xmlDoc = parser.parseFromString(this.baseXml, "application/xml");
      this.root = this.xmlDoc.getElementsByTagName('root')[0];
      this.definitions = jsonYaml.definitions;
      this.buildXml();
    },
    buildXml: function() {
      let classPosition = 1;
      for (let definition in this.definitions) {
        let newClass = this.buildClass(definition, classPosition);
        classPosition++;
      }
      let s = new XMLSerializer();
      console.log(s.serializeToString(this.xmlDoc));
    },
    buildClass: function(definition, position) {
      let style = 'swimlane;fontStyle=0;childLayout=stackLayout;horizontal=1;startSize=26;fillColor=none;horizontalStack=0;resizeParent=1;resizeParentMax=0;resizeLast=0;collapsible=1;marginBottom=0;swimlaneFillColor=#ffffff;';
      let width = 140;
      let spacing = 20;
      let y = 10;
      let fieldNumber = Object.keys(this.definitions[definition].properties).length;
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
      for (let property in this.definitions[definition].properties) {
        let newField = this.buildField(property, iterator, parent);
        iterator++;
      }
      return newClass;
    },
    buildField: function(property, iterator, parent) {
      let style = 'text;strokeColor=none;fillColor=none;align=left;verticalAlign=top;spacingLeft=4;spacingRight=4;overflow=hidden;rotatable=0;points=[[0,0.5],[1,0.5]];portConstraint=eastwest;';
      let value = property;
      let newField = this.xmlDoc.createElement('mxCell');
      newField.setAttribute('id', ++this.currentID);
      newField.setAttribute('value', value);
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
    }
  }
}
</script>

<style scoped>
</style>
