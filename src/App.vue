<template>
   <v-app>
     <v-app-bar app dark>
       <v-toolbar-title>
         Algorytm Dijsktry
       </v-toolbar-title>
     </v-app-bar>
      <v-main>
        <div id="container">
          <div>
            <v-spacer></v-spacer>
            <v-btn dark class="btn_main" v-on:click=makeGraph>
              Generuj graf
            </v-btn>
            <v-btn dark class="btn_main"
            v-on:click=addNode>
              Dodaj Wezel
            </v-btn>
            <dialog_edge_add v-bind:nodes="this.nodes"
            @result='addEdge'/>

            <dialog_edge_edit
            v-bind:edges="this.edges"
            @result='editEdge'/>

           <dialog_node_rm v-bind:nodes="this.nodes"
           @result='removeNode' />

            <dialog_edge_rm v-bind:edges="this.edges"
            @result='removeEdge'/>
          <v-row justify="center">
        </v-row>
            <v-spacer></v-spacer>
            <br />
            <v-btn
            style="margin-top: 30px; margin-right: 150px; background: #569946"

              class="btn_main float-right"
            v-on:click="dijsktra">
              wykonaj algorytm
            </v-btn>
          </div>
          <br />
          <div id="selecty" style="text-align: center; margin-top: 30px">
            <br />
          <div class="select_main">
          Wybierz liczbę wierzchołków
            <v-select
              :items="nodes_number_btn" 
              v-model="nodesNumberSelected"
              label="Liczba wierzcholkow"
              allow-overflow>
            </v-select>
          </div>
           <div  class="select_main">
           Wybierz wierzchołek początkowy
            <v-select
              :items="nodes" 
              v-model="startNode"
              label="Wierzchołek początkowy"
              allow-overflow>
            </v-select>
          </div>
         
          <div  class="select_main">
           Wybierz wierzchołek docelowy
            <v-select
              :items="nodes" 
              v-model="targetNode"
              label="Wierzchołek docelowy"
              allow-overflow>
            </v-select>
          </div>
          <div style="clear: both"></div>

          </div>
          <div id="graph">
            <cytoscape  :config=this.graphConfig
            ref='cGraph'
            >
              <cy-element
                v-for="def in elements"
                :key="`${def.data.id}`"
                :definition="def"
              />
            </cytoscape>
          </div>
          <div id="result">
            <h2>Rezultat wykonania algorytmu:</h2>
            <span v-html="resultDescription"></span>
          </div>
           <div id="desc">
            <h2>Opis wykonania algorytmu:</h2>
            <span v-html="currentStepDesc"></span>
            <v-btn dark bottom right absolute
            :disabled=!this.nextStepButton
            v-on:click="descStep(1)">
              Następny krok
            </v-btn>
             <v-btn dark bottom left absolute
            :disabled=!this.prevStepButton
            v-on:click="descStep(2)">
              Poprzedni krok
            </v-btn>
          </div>
        </div>
    </v-main>
    
   </v-app>
</template>

<script>
import dialog_edge_rm from './components/DialogEdgeRm.vue';
import dialog_node_rm from './components/DialogNodeRm.vue';
import dialog_edge_add from './components/DialogEdgeAdd';
import dialog_edge_edit from './components/DialogEdgeEdit';
export default {
  name: 'App',
  components:{
    dialog_edge_rm,
    dialog_node_rm,
    dialog_edge_add,
    dialog_edge_edit
  },
  data(){
    return{
      nodesNumberSelected: 5,
      stepByStepDesc:[],
      currentStepDesc:"",
      resultDescription:"",
      startNode:"",
      targetNode:"",
      singleNodeConn:2,
      nodes:[],
      edges:[],
      d_dijkstra:[],
      p_dijsktra:[],
      ng_matrix:[],       //neighbourhood matrix
      explanationStep:0,
      nextStepButton:false,
      prevStepButton:false,
      elements: {},
      graphConfig:{
        zoomingEnabled: false,
        userZoomingEnabled:false,
        panningEnabled: false,
        userPanningEnabled: false,
        style: [ // the stylesheet for the graph
          {
            selector: 'node',
            style: {
              'background-color': '#666',
              'label': 'data(id)'
            }
          },
          {
            selector: 'edge',
            style: {
              'width': 3,
              'line-color': '#ccc',
              'target-arrow-color': '#ccc',
              'color': '#111111',
              'font-size': '30',
              'text-valign': 'top',
              'text-halign': 'top',
              'text-background-color': '#ffffff',
              //'target-arrow-shape': 'triangle',
              'curve-style': 'bezier',
              'label': 'data(weight)'
            }
          }
        ],
        layout: {
          name: 'circle'
        }
      },
      test:"test",
      nodes_number_btn:["3","4","5","6","7"],
      single_node_conn:["2","3"]
    }
  },
  computed: {
        minConnNumber: function(){
          return this.nodesNumberSelected/2;
        },
        maxConnNumber: function(){
          return Math.round(+this.nodesNumberSelected+(this.nodesNumberSelected/3))
        }
  },
  methods:{
    generateNodes(nodesNumber, version){
      let controlEdges=[];
     // controlEdges.push(1+''+(nodesNumber-1))
      let graph_nodes=[];
      // add nodes
      for(let i=0;i<nodesNumber;i++){
        this.nodes.push('W'+(i+1))
        graph_nodes[i]={data: {id: 'W'+(i+1)}}
      }
      for(let k in this.nodes){
        console.log(k)
      }
      if(version==1){
         for(let i=1;i<nodesNumber;i++){
          this.edges.push(i+''+(i+1))
          controlEdges.push(i+''+(i+1))
          graph_nodes.push({data: {id:('W'+i+'W'+(i+1)), source: 'W'+i, target: 'W'+(i+1), weight: Math.round((Math.random()*9)+1)}})
        }
        this.edges.push(nodesNumber+''+1)
        controlEdges.push(nodesNumber+''+1)
        graph_nodes.push({data: {id:('W'+nodesNumber+'W1'), source: 'W'+nodesNumber, target: 'W1', weight: Math.round((Math.random()*9)+1)}})
        // add central connections
        let centralEdges=Math.round(Math.random()*(this.maxConnNumber-this.minConnNumber)+this.minConnNumber)%this.nodesNumberSelected
        let flag=false, firstNode, secondNode
        this.edgesNumber=centralEdges+nodesNumber+1;
        for(let j=0;j<centralEdges;j++)
        {
          let counter=0
          do{
            counter++;
            firstNode=Math.round(Math.random()*(nodesNumber-1))+1;
            secondNode=Math.round(Math.random()*(nodesNumber-1))+1;
            (firstNode==secondNode || (+firstNode+1==secondNode))?flag=false: flag=true;
            //check whether that node is already created
            if(flag){
              for(let i in controlEdges){
                if((firstNode+''+secondNode==controlEdges[i])||(secondNode+''+firstNode==controlEdges[i])){
                  flag=false;
                }
              }
            }
            if(counter>100) break;
          }while(!flag)
          if(counter>100) continue;
          controlEdges.push(firstNode+''+secondNode);
          this.edges.push(firstNode+''+secondNode);
          graph_nodes.push({data: {id:('W'+firstNode+'W'+secondNode), source: 'W'+firstNode, target: 'W'+secondNode, weight: Math.round((Math.random()*10)+1)}})
        }
      }else if(version==2){
        let nodesEdgesControl=[];
        for(let k in this.nodes){
          nodesEdgesControl[+k+1]=+0;
        }
          for(let k in this.nodes){
            let source=+k+1;
            let target, flag=false;
            for(let a=0;a<2;a++){
              let counter=0;
              do{
                counter++;
                target=Math.round(Math.random()*(nodesNumber-1)+1);
                (source==target )?flag=false: flag=true;
                //check whether that node is already created
                if(flag){
                  for(let i in controlEdges){
                    // Czy nie istneje juz takie połączenie oraz czy węzeł nie posiada juz maksymalnej liczby 2 połączen
                    if((target+''+source==controlEdges[i])||(source+''+target==controlEdges[i])||nodesEdgesControl[source]>=2||nodesEdgesControl[target]>=2){
                      flag=false;
                    }
                  }
                }
                
                if(counter>99) break;
              }while(!flag)
              if(counter>99) continue;
              nodesEdgesControl[source]++;
              nodesEdgesControl[target]++;
              controlEdges.push(source+''+target);
              this.edges.push(source+''+target);
              graph_nodes.push({data: {id:('W'+source+'W'+target), source: 'W'+source, target: 'W'+target, weight: Math.round(Math.random()*9)+1}})
            }
            }
             
      }
      // add edges connections
     
      return graph_nodes
    },
    makeGraph(){
      this.$refs.cGraph.instance.remove('*');
      this.nodes.length=0;
      this.edges.length=0;
      this.$refs.cGraph.instance.add(this.generateNodes(this.nodesNumberSelected,1));
      this.$nextTick(() => {
         this.$refs.cGraph.instance.layout({name: 'circle'}).run();
        this.$refs.cGraph.instance.fit(null, 200)
      });
      
      
    },
    makeGraphv2(){
      this.$refs.cGraph.instance.remove('*');
      this.nodes.length=0;
      this.edges.length=0;
      this.$refs.cGraph.instance.add(this.generateNodes(this.nodesNumberSelected,2));
      this.$nextTick(() => {
         this.$refs.cGraph.instance.layout({name: 'circle'}).run();
        this.$refs.cGraph.instance.fit(null, 200)
      });
      
      
    },
    addNode(){
     
      this.nodes.push('W'+(+this.nodes.length+1));
      this.$refs.cGraph.instance.add({data: {id: ('W'+this.nodes.length)}});
      this.$nextTick(() => {
         this.$refs.cGraph.instance.layout({name: 'circle'}).run();
      });
    },
    addEdge(value){
     
      let nodes=value[0]
      let edge=this.$refs.cGraph.instance.filter('edge[id="'+nodes+'"]')
      
      if(edge.length==0&&nodes[1]!=nodes[3]){
        this.edges.push(nodes[1]+''+nodes[3]);
        this.$refs.cGraph.instance.add({data: {id: nodes,source: ('W'+nodes[1]), target: 'W'+nodes[3], weight: +value[1]}})
         this.$nextTick(() => {
         this.$refs.cGraph.instance.layout({name: 'circle'}).run();
      });
      }
    },
    editEdge(value){
      let nodes=value[0];
      this.removeEdge(value[0]);
      value[0]='W'+nodes[0]+'W'+nodes[1];
      this.addEdge(value)
    },
    removeNode(value){
      let nodeNumber=value[1];
      let index=this.nodes.indexOf(value);
      this.nodes.splice(index,1);
      for(let i =0;i<this.edges.length;i++){
        let bufor=this.edges[i];
        if(bufor[1]==nodeNumber||bufor[0]==nodeNumber){
          this.edges.splice(i,1)
          i--;
        }
      }
      this.$refs.cGraph.instance.remove('node[id="'+value+'"]')
      this.$nextTick(() => {
         this.$refs.cGraph.instance.layout({name: 'circle'}).run();
      });
    },
    removeEdge(value){
      let index = this.edges.indexOf(value);
      this.edges.splice(index,1);
      let id='W'+value[0]+'W'+value[1]
      this.$refs.cGraph.instance.remove('edge[id="'+id+'"]')
      this.$nextTick(() => {
         this.$refs.cGraph.instance.layout({name: 'circle'}).run();
      });
    },

    
    makeNeighbourList(){
      let result={};
      for(let i =0;i<this.edges.length;i++){
        let bufor=this.edges[i];
        let firstNode=(+bufor[0]-1);
        let secondNode=(+bufor[1]-1);
        if(result[firstNode]){
          result[firstNode]+=''+(secondNode)
        }else{
          result[firstNode]=''+secondNode
        }
        if(result[secondNode]){
          result[secondNode]+=''+(firstNode)
        }else{
          result[secondNode]=''+firstNode
        }
      }
      return result;
    },
    // find and return node from q array with lowest cost take from array d
    findNextDijsktraNode(q,d){
      let minDistance=1001;
      let result = null
      for(let i=0;i<q.length;i++){
        let bufor = q[i]
        if(d[bufor]<minDistance){
          minDistance=d[bufor]
          result=bufor
        }
      }
      return result
    },
    dijsktra(){
      this.stepByStepDesc=[];
      let ngList=this.makeNeighbourList()
      this.ng_matrix=ngList;
      console.log(ngList)
       let s=[],q=[],d={},p={};
       let startNode=this.startNode;
       if(startNode==""||this.targetNode==""||this.targetNode==this.startNode){
         alert("Nieprawidłowo ustawione wierzchołki");
         return;
       }
       let activeNode=startNode;

      for(let i=0;i<this.nodes.length;i++){
        let bufor = this.nodes[i];
        q.push(bufor)
        d[bufor]=1000;
        p[bufor]=(-1);
      }
      let startNodeIndex=q.indexOf(startNode);
      d[startNode]=0;
      p[startNode]=0;
      s.push(q[startNodeIndex])
      q.splice(startNodeIndex,1)
      while(q.length>0){
        let nodeNumber=activeNode[1];
        nodeNumber--;
        if(typeof ngList[nodeNumber] !== 'undefined'){
          for(let i=0;i<ngList[nodeNumber].length;i++)
          {
            let neighbour=ngList[nodeNumber][i];
            neighbour++;
            let weight = this.$refs.cGraph.instance.getElementById('W'+activeNode[1]+'W'+neighbour).data('weight')
            if(isNaN(weight)) weight = this.$refs.cGraph.instance.getElementById('W'+neighbour+'W'+activeNode[1]).data('weight')
            console.log(d['W'+neighbour]+'*******'+(+d[activeNode]+weight))
            if(d['W'+neighbour]>d[activeNode]+weight){
              this.stepByStepDesc.push({desc:'Analizowany wierzchołek - '+activeNode+',<br /> Połączenie z wierzchołkiem W'+neighbour+' <br /> Dotychczasowy koszt dojscia do W'+neighbour+'= '+d['W'+neighbour]+'<br /> nowy koszt dojscia= '+d[activeNode]+' + '+weight+'<br /> Znaleziono nową scieżke optymalna',node: activeNode, edge: 'W'+activeNode[1]+'W'+neighbour});
              d['W'+neighbour]=d[activeNode]+weight
              p['W'+neighbour]=activeNode
            }else{
              this.stepByStepDesc.push({desc:'Analizowany wierzchołek - '+activeNode+',<br /> Połączenie z wierzchołkiem W'+neighbour+' <br /> Dotychczasowy koszt dojscia do W'+neighbour+'= '+d['W'+neighbour]+'<br /> nowy koszt dojscia= '+d[activeNode]+' + '+weight+'<br /> Nie zmieniamy ścieżki',node: activeNode, edge: 'W'+activeNode[1]+'W'+neighbour});
            }
          }
        }else{
         
          alert("Wybrany wierzchołek nie ma połączenia z innymi wierzchołkami")
          return
        }
        activeNode=this.findNextDijsktraNode(q, d)
        s.push(q[q.indexOf(activeNode)])
        q.splice(q.indexOf(activeNode),1)
        
      }
      this.nextStepButton=true;
      this.printDesc(d,p,s)
    },
    colorNodes(p){
      this.$refs.cGraph.instance.nodes().style('background-color', '#666');
      this.$refs.cGraph.instance.edges().style('line-color', '#ccc')
      this.$refs.cGraph.instance.nodes('[id = "'+this.startNode+'"]').style('background-color', 'green');
      this.$refs.cGraph.instance.nodes('[id = "'+this.targetNode+'"]').style('background-color', 'red');
      let currentStep=this.targetNode;
      let nextStep=p[this.targetNode]
      while(nextStep!=this.startNode){
        this.$refs.cGraph.instance.edges('[id = "'+currentStep+nextStep+'"]').style('line-color', 'blue');
        this.$refs.cGraph.instance.edges('[id = "'+nextStep+currentStep+'"]').style('line-color', 'blue');
        currentStep=nextStep;
        nextStep=p[nextStep];
      }
      this.$refs.cGraph.instance.edges('[id = "'+currentStep+nextStep+'"]').style('line-color', 'blue');
      this.$refs.cGraph.instance.edges('[id = "'+nextStep+currentStep+'"]').style('line-color', 'blue');
      // this.$nextTick(() => {
      //    this.$refs.cGraph.instance.layout().run();
      // });
    },
    colorNode(node){
      this.$refs.cGraph.instance.nodes('[id = "'+node+'"]').style('background-color', 'green');
    },
    colorEdge(edge){
      this.$refs.cGraph.instance.edges('[id = "'+edge+'"]').style('line-color', 'blue');
      this.$refs.cGraph.instance.edges('[id = "'+edge[2]+edge[3]+edge[0]+edge[1]+'"]').style('line-color', 'blue');
    },
    printDesc(d,p,s){
      this.d_dijkstra=d;
      this.p_dijsktra=p;
     
      console.log(s);
      this.colorNodes(p);
      this.resultDescription="Wierzchołek początkowy: "+this.startNode+", wierzchołek końcowy: "+ this.targetNode+"<br />"
      let path="->"+this.targetNode+"<br />";
      let nextStep=p[this.targetNode]
      while(nextStep!=this.startNode){
        path="->"+nextStep+path;
        nextStep=p[nextStep];
      }
      path="Scieżka: "+this.startNode+path;
      this.resultDescription+=path;
      this.resultDescription+="Koszt przejscia: "+d[this.targetNode];

      this.currentStepDesc=this.stepByStepDesc[0].desc; // set up first stepbystep description message
    },
    descStep(type){
       this.$refs.cGraph.instance.nodes().style('background-color', '#666');
       this.$refs.cGraph.instance.edges().style('line-color', '#ccc')
       if(type==1){
        if(this.stepByStepDesc.length>1+this.explanationStep){
         this.explanationStep++;
         this.currentStepDesc=this.stepByStepDesc[this.explanationStep].desc;
         this.colorEdge(this.stepByStepDesc[this.explanationStep].edge);
         this.colorNode(this.stepByStepDesc[this.explanationStep].node);
         this.prevStepButton=true;
         }else{
           this.nextStepButton=false;
         }
       }else if(type==2){
         if(0<this.explanationStep){
         this.explanationStep--
         this.currentStepDesc=this.stepByStepDesc[this.explanationStep].desc;
         this.colorEdge(this.stepByStepDesc[this.explanationStep].edge);
         this.colorNode(this.stepByStepDesc[this.explanationStep].node);
         this.nextStepButton=true;
         }else{
           this.prevStepButton=false;
         }
       }
       
     
    }

  
    
  }
  
};


</script>
<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#graph{
  background-color: #2c3e50 ;
  position: absolute;
  text-align: left;
  height: 700px;
  width: 700px;
  margin-left: 50px;
  margin-right: 50px;
}

#result{
  background-color: #2c3e50 ;
  color: white;
  position: relative;
  text-align: left;
  float: right;
  height: 325px;
  width: 580px;
  margin-right: 50px;
  
}
#desc{
  background-color: #2c3e50 ;
  color: white;
  clear: both;
  position: relative;
  text-align: left;
  float: right;
  height: 325px;
  width: 580px;
  margin-top: 50px;
  margin-right: 50px;
  display: block;
}
#selecty{
  margin-left: 50px;
  margin-right: 50px;
  text-align: center;
}

v-btn{
  margin: 6pt;
  float: left;
}

.btn_main{
  margin: 2pt;
}

.select_main{
  width: 300px;
  text-align: center;
  float: left;
  margin: 25px;
}

.right_menu{
  margin-top: 10pt;
  float: left
}


</style>