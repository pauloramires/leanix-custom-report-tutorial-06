# Exporting a Custom Report into PowerPoint
Custom reports are a great way of creating specific views of your LeanIX workspace data. Nevertheless, many times the user needs to communicate those views to stakeholders that may not have direct access to the LeanIX workspace or are just simply offline. Therefore, the ability to share the information in a popular document format such as PowerPoint, for example, allows the user to effectively increase the reach of the Custom Report message into a broader audience.
In this step-by-step tutorial we'll create a [LeanIX](https://www.leanix.net/en/) custom report that demonstrates how to build an editable Business Model Canvas template, save and load it from a JSON file, and export it as a Power Point presentation.
<div  style="display:flex; justify-content:center">
  <img  src="https://i.imgur.com/t87rf7f.png">
</div>

The complete source-code for this project can be found [here](https://github.com/pauloramires/leanix-custom-report-tutorial-06).

## Pre-requisites

*  [NodeJS LTS](https://nodejs.org/en/) installed in your computer.

## Getting started

Install the [leanix-reporting-cli](https://github.com/leanix/leanix-reporting-cli) globally via npm:

```bash
npm install -g @leanix/reporting-cli
```

Initialize a new project:

```bash
mkdir leanix-custom-report-tutorial-06
cd leanix-custom-report-tutorial-06
lxr init
npm install
```
Configure your environment by editing the *lxr.json* file, if required:
```json
{
  "host": "app.leanix.net",
  "apitoken": "your-api-token-here"
}
```

After this procedure, you should end up with the following project structure:

<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/sJBtRD6.png">
</div>

## Adjusting the report boilerplate source code

We need to make some modifications in our project's boilerplate code. We start by adding the following dependencies:
```bash
npm install --dev @babel/plugin-transform-runtime postcss-loader tailwindcss
npm install alpinejs pptxgenjs
```

 **Note:** During the course of this tutorial, we'll be using the [Alpine JS](https://github.com/alpinejs/alpine),  [Tailwind CSS](https://tailwindcss.com/) and [PptxGenJS](https://gitbrent.github.io/PptxGenJS/) libraries.

After installing the dependencies, we modify the *webpack.config.js* file and include the *@babel/plugin-transform-runtime* and the *postcss-loader*, as indicated by the red arrows in the picture below:

<div  style="display:flex; justify-content:center;">
  <img  src="https://i.imgur.com/Vn0ZeWK.png">
</div>

 We then clean up our project source code by deleting the unnecessary files:
-  *src/report.js*
-  *src/fact-sheet-mapper.js*
-  *src/assets/bar.css*
-  *src/assets/main.css*

Next we create a *postcss.config.js* file in the **root** folder of our project, with the following content:
```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('tailwindcss'),
    require('autoprefixer')
  ]
}
```

Additionally we create an *tailwind.css* file in the assets folder with the following content:

```css
/* src/assets/tailwind.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Next you should set the *src/index.html* file with the contents below:
```html
<!-- src/index.html -->
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="application-name" content="leanix-custom-report-tutorial-06">
  <meta name="description" content="Creating a Business Model Canvas and export it into PowerPoint">
  <meta name="author" content="LeanIX GmbH">
  <title>Creating a Business Model Canvas and export it into PowerPoint</title>
  <style>
    [x-cloak] {
      display: none;
    }
  </style>
</head>
<body x-data="initializeContext()" x-init="initializeReport()">
  <div x-cloak class="container mx-auto h-screen">
  </div>
</body>
</html>
```

And finally set the *src/index.js* file content as follows:
```javascript
// src/index.js

import 'alpinejs'
import '@leanix/reporting'
import pptxgen from 'pptxgenjs'
import './assets/tailwind.css'

const  state = {
  fields: [],
  documents: {}
}

const methods = {
  async initializeReport () {
    await lx.init()
    await lx.ready({})
  },
  onFileChange () {
    // to be implemented
  },
  saveFile () {
    // to be implemented
  },
  exportToPPT () {
    // to be implemented
  }
}

window.initializeContext = () => {
  return {
    ...state,
    ...methods
  }
}
```

Your project folder should look now like this:
<div style="display:flex; justify-content:center">
  <img src="https://i.imgur.com/INgo3a3.png">
</div>

You may start the development server now by running the following command:
```bash
npm start
```
**Note!**

When you run *npm start*, a local webserver is hosted on *localhost:8080* that allows connections via HTTPS. But since just a development SSL certificate is created the browser might show a warning that the connection is not secure. You could either allow connections to this host anyways, or create your own self-signed certificate: https://www.tonyerwin.com/2014/09/generating-self-signed-ssl-certificates.html#MacKeyChainAccess.

If you decide to add a security exception to your localhost, make sure you open a second browser tab and point it to https://localhost:8080. Once the security exception is added to your browser, reload the original url of your development server and open the development console. Your should see a screen similar to the one below:
<div  style="display:flex; justify-content:center">
  <img  src="https://i.imgur.com/Jrn3RXQ.png">
</div>

Now that we have the base structure for our Custom Report, let's proceed into its design!

## Custom Report Design

We'll design our Custom Report with 2 sections: the action bar, on top, which will hold three action buttons and the container, on the bottom, for our business model canvas.
<div  style="display:flex; justify-content:center">
  <img  src="https://svgshare.com/i/NpL.svg">
</div>

As a first step, we'll edit the *src/index.html* file and add the basic template of our report by replacing the existing ``body`` tag content by the content indicated below:
```html
<!-- src/index.html -->
(...)
<body x-data="initializeContext()" x-init="initializeReport()">
  <div x-cloak class="container mx-auto h-screen flex flex-col p-8">

    <!-- the Action Bar container -->
    <div class="mb-4 flex justify-end gap-1">

      <!-- the "Load" button -->
      <label>
        <span class="cursor-pointer inline-flex items-center px-2 py-1 border border-transparent text-xs leading-4 font-semibold tracking-wide rounded text-white bg-red-600 hover:bg-red-500 transition ease-in-out duration-150">
          Load
        </span>
        <input @change="onFileChange" type="file" class="hidden" accept=".json">
      </label>

      <!-- the "Save" button -->
      <span class="inline-flex rounded-md shadow-sm">
        <button @click="saveFile" type="button"
        class="inline-flex items-center px-2 py-1 border border-transparent text-xs leading-4 font-semibold tracking-wide rounded text-white bg-green-600 hover:bg-green-500 transition ease-in-out duration-150">
          Save
        </button>
      </span>

      <!-- the "Export to PPT" button -->
      <span class="inline-flex rounded-md shadow-sm">
        <button @click="exportToPPT" type="button"
          class="inline-flex items-center px-2 py-1 border border-transparent text-xs leading-4 font-semibold tracking-wide rounded text-white bg-indigo-600 hover:bg-indigo-500 transition ease-in-out duration-150">
          Export to PPT
        </button>
      </span>

    </div>

    <!-- the Business Model Canvas container -->
    <div
      class="grid h-full border-t border-r rounded border-gray-400 text-gray-800 text-sm font-semibold bg-gray-400"
      x-ref="container">
    </div>
  </div>
</body>
(...)
```
Notice that the "Save" and "ExportPPT" buttons have listeners for the [@click](https://github.com/alpinejs/alpine#x-on) event that trigger the "saveFile" and "exportToPPT" methods respectively, whereas the "Load" button listens to [@change](https://github.com/alpinejs/alpine#x-on) event that triggers the "onFileChange" method. Since we have previously created empty placeholders for those methods  in the *methods* object of our *src/index.js* file, we'll implement them ahead in this tutorial.

Your report should look like this now:
<div  style="display:flex; justify-content:center">
  <img  src="https://i.imgur.com/QUxjN3s.png">
</div>



###  The Business Model Canvas grid
For modelling the Business Model Canvas template in our Custom Report, we'll use a [CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout) of 10 columns by 3 rows, as depicted below.

<div  style="display:flex; justify-content:center">
  <img  src="https://svgur.com/i/NpA.svg">
</div>

In order to place and size correctly the Business Model Canvas fields in our grid we'll define, for each field, its origin and span expressed in terms of columns and rows:

<div  style="display:flex; justify-content:center">
  <img  src="https://svgur.com/i/NmM.svg">
</div>

[Tailwind CSS](https://tailwindcss.com/) provides a set of grid utility classes - the [Grid Column Start/End](https://tailwindcss.com/docs/grid-column) and the [Grid Row Start/End](https://tailwindcss.com/docs/grid-row),  that are used to set the fields on the canvas. The table below summarizes, for each field, its grid coordinates,  span, and the [Tailwind CSS](https://tailwindcss.com/docs/grid-row) utility classes used to place and size it on the grid:
| Field | Column Start | Column Span | Row Start | Row Span | Grid utility classes |
| :-------------: |-------------:| -----:|  -----:|  -----:| :-----:|
| Key Partners | 1 | 2 | 1 | 2 | *"col-start-1 col-span-2 row-start-1 row-span-2"*
| Key Activities | 3 | 2 | 1 | 1 | *"col-start-3 col-span-2 row-start-1 row-span-1"*
| Key Resources | 3 | 2 | 2 | 1 | *"col-start-3 col-span-2 row-start-2 row-span-1"*
| Value Propositions | 5 | 2 | 1 | 2 | *"col-start-5 col-span-2 row-start-1 row-span-2"*
| Customer Relationships | 7 | 2 | 1 | 1 | *"col-start-7 col-span-2 row-start-1 row-span-1"*
| Channels | 7 | 2 | 2 | 1 | *"col-start-7 col-span-2 row-start-2 row-span-1"*
| Customer Segments | 9 | 2 | 1 | 2 | *"col-start-9 col-span-2 row-start-1 row-span-2"*
| Cost Structure | 1 | 5 | 3 | 1 | *"col-start-1 col-span-5 row-start-3 row-span-1"*
| Revenue Streams | 6 | 5 | 3 | 1 | *"col-start-6 col-span-5 row-start-3 row-span-1"*

In order to render those fields programmatically in our HTML template, we'll define the **fields** array in the state variable of the *src/index.js* file containing each individual field information such as an unique field key, a label to be displayed, and the respective styling classes:
```javascript
// src/index.js
(...)
const  state = {
  fields: [
    { key: 'keyPartners', label: 'Key Partners', classes: 'col-start-1 col-span-2 row-start-1 row-span-2' },
    { key: 'keyActivities', label: 'Key Activities', classes: 'col-start-3 col-span-2 row-start-1 row-span-1' },
    { key: 'keyResources', label: 'Key Resources', classes: 'col-start-3 col-span-2 row-start-2 row-span-1' },
    { key: 'valuePropositions', label: 'Value Propositions', classes: 'col-start-5 col-span-2 row-start-1 row-span-2' },
    { key: 'customerRelationships', label: 'Customer Relationships', classes: 'col-start-7 col-span-2 row-span-1' },
    { key: 'channels', label: 'Channels', classes: 'col-start-7 col-span-2 row-span-1' },
    { key: 'customerSegments', label: 'Customer Segments', classes: 'col-start-9 col-span-2 row-start-1 row-span-2' },
    { key: 'costStructure', label: 'Cost Structure', classes: 'col-span-5 row-start-3 row-span-1' },
    { key: 'revenueStreams', label: 'Revenue Streams', classes: 'col-span-5 row-start-3 row-span-1' }
  ],
  document: {}
}
(...)
}
```
Furthermore we'll add also, in our *src/index.html* file a template that renders  in our business model canvas container grid, recursively, each field defined previously:
```html
<!-- src/index.html -->
(...)
    <!-- the Business Model Canvas container -->
    <div
      class="grid h-full border-t border-r rounded border-gray-400 text-gray-800 text-sm font-semibold"
      x-ref="container">
      <!-- recursive template for the grid fields -->
      <template x-for="field in fields" :key="field.key">
        <!-- the field container -->
        <div
          :field="field.key"
          :class="field.classes"
          class="border-l border-b border-gray-400 p-2 flex flex-col">
          <!-- the field label -->
          <div
            field-label
            class="px-1 text-base mb-1 text-gray-700 truncate"
            x-text="field.label">
          </div>
          <!-- the field input textarea, editable by the user -->
          <textarea
            field-content
            x-model="document[field.key]"
            class="text-sm tracking-wide bg-gray-100 hover:bg-gray-200 focus:bg-gray-200 transition-color duration-250 w-full flex-1 border border-dotted rounded p-2">
          </textarea>
        </div>
      </template>
    </div>
(...)
```
You should see the Business Model Canvas template rendered correctly:
<div  style="display:flex; justify-content:center">
  <img  src="https://i.imgur.com/wTewxyB.png">
</div>

We have now implemented our Custom Report design, both the action bar containing the Load, Save and Export to PPT buttons and the Business Model Canvas container grid. However, we are still missing the business logic required to implement all the import and export functionality required for this custom report. We'll cover that next!

## Business Logic
We are looking to provide to this Custom Report three main functionalities: exporting the Business Model contents as a JSON file, importing the contents from a  JSON file, and exporting the whole Business Model Canvas as a Power Point slide. In this chapter we'll cover the implementation of each individually.

### Exporting the Business Model Canvas content as a JSON file
The first functionality that we'll cover is the export method of our Business Model Canvas content as a JSON file. The basic idea here is that all Business Model Canvas content is stored under the state variable **document** previously defined in our *src/index.js* file. When the user updates the content of any field, that content is automatically set as an attribute of  the **document** object under the corresponding field's key. Therefore, the state of the Business Canvas Model can be exported any time by simply serializing the content of the **document** variable as JSON, and saving it as a text file into the user's local filesystem. Given below is an implementation of this process, which should be copied into the **saveFile** method placeholder previously defined in our *src*/index.js* file:
```javascript
// src/index.js
(...)
const methods = {
  (...)
  saveFile () {
    const data = JSON.stringify(this.document, null, 2)
    const blob = new  Blob([data], { type: 'text/plain' })
    const e = document.createEvent('MouseEvents')
    const a = document.createElement('a')
    a.download = 'businessModelCanvas.json'
    a.href = window.URL.createObjectURL(blob)
    a.dataset.downloadurl = ['text/json', a.download, a.href].join(':')
    e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null)
    a.dispatchEvent(e)
  },
 (...)
}
```
Now if you fill a couple of fields in your report and click on the green "Save" button, you should be able to save them as a JSON file:
<div  style="display:flex; justify-content:center">
  <img  src="https://i.imgur.com/yyonhAo.png">
</div>

### Loading JSON data
Having the JSON export feature covered, we'll now turn on how to import a JSON file into our Custom Report. As we have seen, all of our Business Model Canvas fields content is stored in the **document** variable. Therefore, our import method will replace the content of that variable by the content of the json file selected by user. In order to allow the user to browse and select a file from the local storage, the *Load* button uses a hidden [input field of the type *file*](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file). Once the user clicks on it and chooses a file, it will trigger the **onFileChange** method. Since we have previously defined the empty placeholder for the **onFileChange** method in our *src/index.js* file, we fill it in with the example implementation given below:

```javascript
// src/index.js
(...)
const methods = {
  (...)
  onFileChange (evt) {
    let files = evt.target.files || evt.dataTransfer.files
    if (!files.length) return
    let reader = new FileReader()
    reader.onload = e => { this.document = JSON.parse(e.target.result) }
    reader.readAsText(files[0])
  },
 (...)
}
```
Now if you click on the red *Load* button you should be able to restore any previously saved Business Model Canvas json file. Try it!

### Exporting to PowerPoint
Finally, the last method we'll cover in this chapter relates to the exportation of our Business Model Canvas as a PowerPoint presentation. This method is a bit more elaborated than the two previous methods covered before since it requires not only to capture the content of our Business Model Canvas fields, but their geometry as well so to render them properly in our presentation.
For this section we'll be using the popular [PptxGenJs](https://gitbrent.github.io/PptxGenJS/) library, which has an extensive and well documented API. This library allows the developer to programmatically create rich PowerPoint presentations with charts, images, shapes, text and tables among other elements. For our use case, we'll be focusing on two specific methods of this api: [adding shapes](https://gitbrent.github.io/PptxGenJS/docs/api-shapes.html) and [adding text](https://gitbrent.github.io/PptxGenJS/docs/api-text.html).

By design, the [PptxGenJs](https://gitbrent.github.io/PptxGenJS/) library currently works with two types of units for positioning and sizing shapes and text: inches and percentages of the slide dimensions. In order to scale properly the geometry of our Business Model Canvas template from HTML into the PowerPoint slide, the percentage unit becomes more convenient. Therefore, we capture the origin coordinates (x and y), width and height of our Business Model Canvas container element, in pixels, and express the geometries of its children elements - the fields, in terms of normalized percentages relatively to its parent container. Moreover, we'll make this Business Model Canvas container element scale to fit to the slide geometry by assigning its width and height, in slide dimension percentage terms, to 100%. Furthermore, and knowing that each child element contains two text fields - the field label and the field content, we need to capture also, for each, their normalized bounding box - giving their position and sizing, as well as its text content.
Given below is the documented implementation that should be copied into the empty **exportToPPT** placeholder of our *src/index.js* file:

```javascript
// src/index.js
(...)
const methods = {
  (...)
  exportToPPT () {
    // get an handle to our Business Model Canvas container element
    const containerEl = this.$refs.container
    // get the origin coordinates - x0, y0, width and height of it
    const { x: x0, y: y0, width: containerWidth, height: containerHeight } = containerEl.getBoundingClientRect()

    // auxiliar method for normalizing an element geometry relatively
    // to our business model canvas container, in terms of percentage
    const getNormalizedElBbox = el  => {
      let { x, y, width, height } = el.getBoundingClientRect()
      const  bbox = {
        x: ((x - x0) / containerWidth) * 100,
        y: ((y - y0) / containerHeight) * 100,
        width: (width / containerWidth) * 100,
        height: (height / containerHeight) * 100,
      }
      // round the values of our bbox object attributes to decimal places
      // and append to them a '%' character, as required by the PptxGenJS API
      const normalizedBbox = Object.entries(bbox)
        .reduce((accumulator, [key, value]) => ({ ...accumulator, [key]:  value.toFixed(2) + '%'}), {})
      return normalizedBbox
    }

    // For each Business Model Canvas container field, marked with the directive 'field'
    const fields = Array.from(containerEl.querySelectorAll('[field]'))
      .map(fieldEl  => {
	// get the normalized geometry and shape attributes of its outer container
        const containerBbox = {
          ...getNormalizedElBbox(fieldEl),
          line: { line:  '000000', lineSize:  '1' }
        }
	// get an handle to the field label, marked with the 'field-label' directive
        const  labelEl = fieldEl.querySelectorAll('[field-label')[0]
        // extract its text content
        let { textContent: text = '' } = labelEl
        // get the normalized geometry and text attributes of its content
        const labelBbox = {
          ...getNormalizedElBbox(labelEl),
          textOpts: { autoFit:  true, fontSize:  7, bold:  true, align:  'left', valign:  'top' },
          text
        }

        // get an handle to the field content, marked with the 'field-content' directive
        const contentEl = fieldEl.querySelectorAll('[field-content')[0]
        // extract its value
        text = contentEl.value || ''
        // get the normalized geometry and text attributes of its content
        const contentBbox = {
          ...getNormalizedElBbox(contentEl),
          textOpts: { autoFit: true, fontSize: 7, align: 'left', valign: 'top' },
          text
        }
        // return an array representing the field's container, label and content geometries
        return [containerBbox, labelBbox, contentBbox]
      })

    // create a new presentation
    const pres = new pptxgen()
    // add a slide to the presentation
    const slide = pres.addSlide()
    // for each mapped field of our business model canvas
    fields.forEach(field  => {
      field
        // add a shape if the section corresponds to the field container
        // or a text if the section corresponds to the field's label or content
        .forEach(section => {
          const { x, y, width: w, height: h, text, line = {}, textOpts = {} } = section
          const { rect: shapeType } = pres.ShapeType
          const shapeOpts = {x, y, w, h, ...line, ...textOpts }
          typeof text === 'string'
            ? slide.addText(text, { shape:  shapeType, ...shapeOpts })
            : slide.addShape(shapeType, shapeOpts)
        })
    })
    // and finally save the presentation
    pres.writeFile('BusinessModelCanvas.pptx')
  }
  (...)
}
```

Now if you fill a couple of fields in your report and click on the purple "Export to PPT" button, you should be able to download a powerpoint file which looks like the picture below:
<div  style="display:flex; justify-content:center">
  <img  src="https://i.imgur.com/ZIeTpns.png">
</div>


There it is, your Business Model Canvas custom report is not able to be exported into a PowerPoint presentation. Good work!

## Conclusions and next steps
In this tutorial we have covered a way of exporting a LeanIX Custom Report into a popular portable document format such as PowerPoint. We picked up a popular grid report - the Business Model Canvas, and provided it with basic import and export functionality of both data and layout. As a great follow-up exercise for the reader, we would recommend to add, to this example custom report, an additional export functionality into another popular format such as SVG or PDF.
Congratulations for having completed this tutorial!