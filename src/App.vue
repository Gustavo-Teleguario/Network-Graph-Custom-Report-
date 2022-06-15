<script setup>
import '@leanix/reporting'
import {computed, reactive, ref, watch} from 'vue'
//import * as nNG from "v-network-graph"
import * as vNG from "v-network-graph"
import {ForceLayout,} from "v-network-graph/lib/force-layout"

// reactive variable that will hold the workspace factsheet types
const factSheetTypes = ref([])
// reactive variable will hold the selected factsheet type
const selectedFactSheetType = ref(null)
// reactive variable that will hold the computed averageCompletion statistic
const averageCompletion = ref(null)
//Fact Sheet type List
let list = []
//Für meine Query Application to Interfaces
const relationAppToInter = ref([]);
//Application
const applications = ref([])
//Interfaces
const interfaceVa = ref([])
//Interface
const justInterface = ref([])
// variable for storing the workspace's application lifecycle phases view model
const applicationLifecyclePhases = ref({})

//ARRAY STRING
const justName = ref([])
const queryConsumerApplicationToInterface = `relConsumerApplicationToInterface {
            edges {
              node {
                factSheet {
                  id
                  name
                }
              }
            }
          }`
const queryProviderApplicationToInterface = `relProviderApplicationToInterface {
  edges {
    node {
      factSheet {
        id
        name
      }
    }
  }
}`
// we define our report initialization method, it is an async one so that we
// can syncronously await for Promise results

// the report initialization method
const initializeReport = async () => {
  const setup = await lx.init()
  // we extract here the list of all factsheet types available in the workspace and store it
  factSheetTypes.value = Object.keys(setup.settings.dataModel.factSheets)
  //console.log("Show all FactSheetTyps", factSheetTypes.value)
  // and select the first factsheet type of that list
  selectedFactSheetType.value = factSheetTypes.value?.[0] || null
  /*for (var i = 0; i < factSheetTypes.value.length; i++) {
    list.push(factSheetTypes.value[i])
  }*/
  const config = {
    menuActions: {
      // we enable here the standard "Settings" button
      showConfigure: true,
      // and set the callback for opening the configuration modal defined ahead
      configureCallback: openReportConfigurationModal,
    },
    facets: [
      {
        key: 'main',
        fixedFactSheetType: 'Application',
        attributes: ['id', 'name', queryConsumerApplicationToInterface, queryProviderApplicationToInterface],
        callback: MyFunction.bind(this)
      },
      {
        key: 1,
        fixedFactSheetType: 'Interface',
        attributes: ['id', 'name'],
        callback: MyFunctionInter.bind(this)
      }
    ],
  }
  lx.ready(config)
}

const countAppWithConsumerInterface = ref(null)
const countAppWithProviderInterface = ref(null)

const numberOfAllNodes = ref(null)
/**
 *
 * @param data is responsible for obtaining the data of each Fact Sheet according to its configuration
 * In this case it is all the Elements of all FactSheets Siehe FixedFactedSheetType
 * @constructor
 */
const MyFunction = (data) => {
  applications.value = data
  let countApplication = 0
  applications.value.forEach(elements => {
    //Only Application with relation to Interfaces
    if (elements.relConsumerApplicationToInterface.edges.length != 0
        || elements.relProviderApplicationToInterface.edges.length != 0) {
      countApplication += 1
    }
  })
  // console.log("Count Application ", countApplication)

  let countConsumer = 0
  applications.value.forEach(elements => {
    //GET all Consumer app to Interfaces
    if (elements.relConsumerApplicationToInterface.edges.length != 0) {
      let testingVariable
      testingVariable = elements.relConsumerApplicationToInterface.edges.map(({node}) => node.factSheet.name)
      // console.log("Application Data ", elements.relConsumerApplicationToInterface.edges.map(({node}) => node.factSheet.name))
      for (let i = 0; i < testingVariable.length; i++) {
        //console.log("Rel Consumer App to  Inter ", testingVariable[i].split('>>')[0])
      }
      countConsumer += 1
    }
  });
  countAppWithConsumerInterface.value = countConsumer;


  console.log("Count Consumer ", countConsumer)
  //GET all Provider app To Interface
  //TOTAL 24 NODES
  let countProvider = 0
  applications.value.forEach(el => {
    if (el.relProviderApplicationToInterface.edges.length != 0) {
      let getName
      getName = el.relProviderApplicationToInterface.edges.map(({node}) => node.factSheet.name)
      for (let i = 0; i < getName.length; i++) {
        // console.log("Rel Provider App to  Inter ", getName[i].split('>>')[0])
        countProvider += 1
      }

    }
  })
  countAppWithProviderInterface.value = countProvider
  console.log("Count Provider ", countProvider)
}
const MyFunctionInter = (dataInter) => {
  interfaceVa.value = dataInter;
  //console.log("Interface Data ", interfaceVa.value)
}
// method for opening up the configuration modal and show a single select input containing the list of available factsheet types
const openReportConfigurationModal = async () => {
  const fields = {
    factSheetType: {
      type: 'SingleSelect',
      label: 'FactSheet Type',
      options: factSheetTypes.value
          .map(factSheetType => ({
                value: factSheetType,
                label: lx.translateFactSheetType(factSheetType)
              })
          )
    }
  }
  const initialValues = {factSheetType: selectedFactSheetType.value}
  const values = await lx.openFormModal(fields, initialValues)
  if (values) selectedFactSheetType.value = values.factSheetType
}

// method that will query the workspace for the completion value of the selectedFactSheetType
const fetchGraphQLData = async () => {
  const interfaceComponents = `{
  allFactSheets(factSheetType: Interface) {
    totalCount
    edges {
      node {
        id
        name
        }
      }
    }
  }`
  try {
    lx.showSpinner()
    /* averageCompletion.value = await lx.executeGraphQL(query, {factSheetType: selectedFactSheetType.value})
         .then(({allFactSheets}) => {
           const completionSum = allFactSheets.edges.reduce((accumulator, {node}) => accumulator += node.completion.completion, 0)
           const factSheetCount = allFactSheets.edges.length
           const averageCompletion = completionSum / factSheetCount
           return averageCompletion
         })*/
    //This Query is not necesary
    /* relationAppToInter.value = await lx.executeGraphQL(queryAppToInt)
         .then(data => data.allFactSheets.edges.map(({node}) => node))
     //JUST APPLICATIONS
     console.log("DATA ZEIGEN ", relationAppToInter.value)*/


    //Hier ist eine beispiel wie man Proxy into Proxy noch mal zugreifen kann
    //console.log("RELATION", relationAppToInter.value[1].relConsumerApplicationToInterface.edges.map(({node}) => node.factSheet.name))
    //JUST INTERFACE
    justInterface.value = await lx.executeGraphQL(interfaceComponents)
        .then(data => data.allFactSheets.edges.map(({node}) => node))
  } finally {
    lx.hideSpinner()
  }
}
/***
 *Implementation for the Network-Graph
 ***/

const getNodes = () => {

  const nodes = {}
  const obj = reactive(nodes)
  /**
   * Obtain all application from Fact Sheet type Application, and create the Vertices for
   * each app with Relation to Consumer Interface and Provider Interface. each application is unique and there are no duplicates
   * with the trim method we remove the space in a string
   */
  const nodeName = new Set();
  applications.value.forEach(elements => {
    //Only Application with relation to Interfaces
    if (elements.relConsumerApplicationToInterface.edges.length != 0
        || elements.relProviderApplicationToInterface.edges.length != 0) {
      const nodeId = elements.name.trim()
      nodeName.add(nodeId)
      //const name = elements.name
      //obj[nodeId] = {name};

    }
  });

  /**
   *From Interface pick up just the Application name
   */
  const objConsumerInterface = reactive(obj)
  applications.value.forEach(el => {
    if (el.relConsumerApplicationToInterface.edges.length != 0) {
      let arrayStringNames
      arrayStringNames = el.relConsumerApplicationToInterface.edges.map(({node}) => node.factSheet.name)
      // console.log("Application Data ", elements.relConsumerApplicationToInterface.edges.map(({node}) => node.factSheet.name))
      for (let i = 0; i < arrayStringNames.length; i++) {
        //console.log("Application Data ", arrayStringNames[i].split('>>')[0])
        const nodeId = arrayStringNames[i].split('>>')[0].trim();
        nodeName.add(nodeId)
        //const name = arrayStringNames[i].split('>>')[0];
        //objConsumerInterface[nodeId] = {name};
      }
    }
  });

  const objProviderInterface = reactive(obj)
  applications.value.forEach(el => {
    if (el.relProviderApplicationToInterface.edges.length != 0) {
      let arrayStringNames
      arrayStringNames = el.relProviderApplicationToInterface.edges.map(({node}) => node.factSheet.name)
      //console.log("Provider ", arrayStringNames)
      for (let i = 0; i < arrayStringNames.length; i++) {
        const nodeId = arrayStringNames[i].split('>>')[0].trim();
        //console.log("Application Data ", arrayStringNames[i].split('>>')[0])
        //const name = arrayStringNames[i].split('>>')[0];
        nodeName.add(nodeId)
        //objProviderInterface[nodeId] = {name};
        // count += 1
      }
    }
  });

  /**
   * this list contains only one real name, Set allowed not duplicate
   */
  let countAllNodes = 0;
  nodeName.forEach(el => {
    const name = el
    nodes[name] = {name}
    countAllNodes += 1
    //console.log("ECHTE KNOTEN", name + "All Nodes "+countAllNodes)

  })
  numberOfAllNodes.value = countAllNodes
  return nodes
}
const getEdges = () => {

  const edges = {}
  var count = 1
  const edgesObject = reactive(edges)
  /**
   * Find the relation Consumer from Application to Interface an create just for them a Edge
   * Create Edges from Application to Consumer Interfaces
   */
  let sourceName
  applications.value.forEach(elements => {
    const getConsumerInterfaceObject = ref([])
    if (elements.relConsumerApplicationToInterface.edges.length != 0) {
      sourceName = elements.name.trim();
      //application name from interface and the length for eacht application for Edges
      getConsumerInterfaceObject.value = elements.relConsumerApplicationToInterface.edges.map(({node}) => node.factSheet)
      // appIDFromInterface.value = elements.relConsumerApplicationToInterface.edges.map(({node}) => node.factSheet)
      //console.log("Edge pro Application ", elements.relConsumerApplicationToInterface.edges.length)
      //console.log("Edge source ", nameSource)
      //console.log("CONTENT-CONSUMER ", getConsumerInterfaceObject.value)
      let source = `${sourceName}`;
      for (let x = 0; x < getConsumerInterfaceObject.value.length; x++) {
        let target = `${getConsumerInterfaceObject.value[x].name.split('>>')[0].trim()}`
        const edgeId = `edge${getConsumerInterfaceObject.value[x].id}`
        // const edgeId = `${appIDFromInterface.value[x].id}`
        edgesObject[edgeId] = {source, target}
        //count += 1
      }
    }
  })

  /**
   * Create a Edges from App to Provider Interfaces
   * @type {UnwrapNestedRefs<UnwrapNestedRefs<{}>>}
   */
  const edgesProvider = reactive(edges)
  applications.value.forEach(elements => {
    const getProviderInterfaceObject = ref([])
    if (elements.relProviderApplicationToInterface.edges.length != 0) {
      //app name for Source
      sourceName = elements.name.trim();
      //application name from interface and the length for eacht application for Edges
      getProviderInterfaceObject.value = elements.relProviderApplicationToInterface.edges.map(({node}) => node.factSheet)
      //console.log("Edge pro Application ", elements.relConsumerApplicationToInterface.edges.length)
      //console.log("Edge source ", nameSource)
      //console.log("Edge Target ", appNameFromInterface.value)
      // console.log("CONTENT PROVIDER ", getProviderInterfaceObject.value)

      let source = `${sourceName}`;
      for (let x = 0; x < getProviderInterfaceObject.value.length; x++) {
        let target = `${getProviderInterfaceObject.value[x].name.split('>>')[0].trim()}`
        //const edgeId = `${appIDFromInterface.value[x].id}`
        const edgeId = `edge${count}`
        //const edgeId = `${appIDFromInterface.value}`
        edgesProvider[edgeId] = {source, target}
        count += 1;
      }
    }
  })
  return edges
}

const getConfigs = () => {
  let ancho = 3
  const configs = vNG.defineConfigs({
    view: {
      layoutHandler: new ForceLayout({
        //positionFixedByDrag: true,
        // positionFixedByClickWithAltKey: true,
        //Das hier funktioniert muss man noch mal über d3 Force in google mehr wissen
        createSimulation: (d3, nodes, edges) => {
          //const forceLink = d3.forceLink<ForceEdgeDatum>(edges).id(d => d.id)
          const forceLink = d3.forceLink(edges).id(d => d.id)
          var width = 100;
          var height = 990 * -1;
          var dimension = (100 * 2.2);
          return d3
              .forceSimulation(nodes)
              .force("link", forceLink.distance(6))
              //forceLink.distance(-10))
              //.force("charge", d3.forceManyBody())
              .force("collide", d3.forceCollide(dimension).strength(0.3))
              .force("center", d3.forceCenter(width, height))
              .alphaMin(0.3)
          //.force("link", d3.forceLink().id(d => d.id).distance(1))
          //
        }
      }),
    },
    node: {
      selectable: false,
      normal: {
        radius: 20,
        color: "#8888ff",
        strokeWidth: 2,
        strokeColor: "#0000aa",
      },
      hover: {
        color: "#6666ff",
      },
      label: {
        fontSize: 15,
        color: "#000000",
      },

    },
    edge: {
      //selectable: true,
      normal: {
        width: ancho,
        //ancho > 5 ?  2 : 10,
        color: "#008CBA",
      },
      hover: {
        width: 1,
        color: "#0000aa",
      },

      /* gap: 20,
       type: "curve",
       margin: 6,
       marker: {
         target: {
           type: "circle",
           width: 8,
           height: 8,
         },
       },*/
    },
  })
  return configs
}

// computed variable for translating the selected factsheet type
const factSheetTypeLabel = computed(() => selectedFactSheetType.value !== null
    ? lx.translateFactSheetType(selectedFactSheetType.value, 'plural')
    : null)


// watcher that will trigger the fetchGraphQLData on every selectedFactSheetType update
watch(selectedFactSheetType, fetchGraphQLData)
// watcher that will trigger the updateChart method on every averageCompletion variable update
// we call our report initialization method here...

initializeReport()
</script>

<template>
  <!--<div :style="{color: ' #0000FF'}">-->
  <div style="background: Lavender; font-size: 20px; padding: 10px; border: 1px solid lightgray; margin: 10px;">
    <h1>Network-Graph Custom Report</h1>
    <h2 style="font-size:18px;">Visualizing relation between Applications containing either a Provider or Consumer
      Interface </h2>
    <button class="button">Application with Consumer Interfaces = {{ countAppWithConsumerInterface }} </button>
    <button class="button button2">Application with Provider Interfaces = {{ countAppWithProviderInterface }}</button>
    <button class="button button3">Total number of all Application = {{numberOfAllNodes}}</button>
  </div>
  <v-network-graph
      :zoom-level="0.3"
      :nodes=getNodes()
      :edges=getEdges()
      :configs=getConfigs()
  />
</template>
<style>
.button {
  background-color: #00B2EE; /* Green */
  border: none;
  color: white;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
}

.button2 {background-color: #008CBA;} /* Blue */
.button3 {background-color: #6050DC;} /* Red */
</style>