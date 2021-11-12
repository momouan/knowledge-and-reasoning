<template>
    <!-- <div v-for="(g, index) in grid" :key="`g-${index}`">
        <div v-for="(cell, index) in g.cells" :key="`cell-${index}`">
            <h4> i am inside </h4>
        </div>
    </div> -->
    <div class="container">
      <button @click="init()">INIT THE SHIT</button>
      <div>
      <button id="up" @click="moveNorth()">Up</button>
      <button id="down" @click="moveSouth()">Down</button>
      <button id="right" @click="moveEast()">Right</button>
      <button id="left" @click="moveWest()">Left </button>
      </div>
    <table>
    <tr v-for="(i, ind1) in 10" :key="`i-${ind1}`" :id="`${i-1}`">
        <td v-for="(j, ind2) in 10" :key="`j-${ind2}`" :id="`${i-1}${ind2}`">   
        </td>
    </tr>
    </table>
    </div>
</template>

<script>
const { Connection, query } = require('stardog');
// import Stardog from 'stardog-js';
// const stardog = new Stardog({
//     endpoint: 'http://localhost:5820',
//     database: 'proj_kr',
//     auth: {
//         user: 'admin',
//         pass: 'admin'
//     }
// });
export default {
  
  name: 'Grid',
  data(){
      return {
        direction: "IsEast",
    //       cellID: 1,
    //     grid: [
    //   {
    //     cells: ['1.1', '1.2']
    //   },
    //   {
    //     cells: ['2.1', '2.2']
    //   }
    // ]
      }
  },
  props: {
    
  },
  methods: {
     init(){
     query.execute(conn, 'proj_kr', 'SELECT ?c WHERE { ?c a :CellPlayer. }', 
                  'application/sparql-results+json', {
      reasoning: true
    }).then(({ body }) => { 
      body = body.results.bindings[0].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
      document.getElementById(body).textContent = "Player"
    });
    query.execute(conn, 'proj_kr', 'SELECT ?c WHERE { ?c a :Ball. }', 
                  'application/sparql-results+json', {
      reasoning: true
    }).then(({ body }) => { 
      body = body.results.bindings[0].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
      document.getElementById(body).textContent = "Ball"
    });
    // (async () => {
    //     let data = stardog.query({
    //     query: 'select ?c where {?c a :CellPlayer.}',
    //     // graph: 'tag:stardog:api:context:default',
    //     // offset: 0,
    //     // limit: 1,
    //     // timeout: 1000,
    //     reasoning: true
    //     });

    // });
   },
   async moveSouth(){
       await query.execute(conn, 'proj_kr', 'DELETE { ?c a :CellPlayer. ?ball a :Ball } INSERT { ?x a :CellPlayer. ?newBall a :Ball. }WHERE{{ ?c a :CellPlayer. ?x a :IsSouth. FILTER NOT EXISTS { ?x a :Ball .}} UNION { ?c a :CellPlayer. ?x a :IsSouth. ?ball a :IsSouth. ?ball a :Ball. ?ball :hasSouth ?newBall. ?newBall a :NotWall. }}', 
                  'application/sparql-results+json', {
      reasoning: true
    })
    for (let i = 0; i < 10; i++) {
            for (let j = 0; j < 10; j++) {
                document.getElementById(i + '' + j).textContent = null       
            }
        }
    this.init()
    },
   async moveNorth(){
       await query.execute(conn, 'proj_kr', 'DELETE { ?c a :CellPlayer. ?ball a :Ball } INSERT { ?x a :CellPlayer. ?newBall a :Ball. }WHERE{{ ?c a :CellPlayer. ?x a :IsNorth. FILTER NOT EXISTS { ?x a :Ball .}} UNION { ?c a :CellPlayer. ?x a :IsNorth. ?ball a :IsNorth. ?ball a :Ball. ?ball :hasNorth ?newBall. ?newBall a :NotWall. }}', 
                  'application/sparql-results+json', {
      reasoning: true
    })
    for (let i = 0; i < 10; i++) {
            for (let j = 0; j < 10; j++) {
                document.getElementById(i + '' + j).textContent = null       
            }
        }
    this.init()
    },
   async moveEast(){
       await query.execute(conn, 'proj_kr', 'DELETE { ?c a :CellPlayer. ?ball a :Ball } INSERT { ?x a :CellPlayer. ?newBall a :Ball. }WHERE{{ ?c a :CellPlayer. ?x a :IsEast. FILTER NOT EXISTS { ?x a :Ball .}} UNION { ?c a :CellPlayer. ?x a :IsEast. ?ball a :IsEast. ?ball a :Ball. ?ball :hasEast ?newBall. ?newBall a :NotWall. }}', 
                  'application/sparql-results+json', {
      reasoning: true
    })
    for (let i = 0; i < 10; i++) {
            for (let j = 0; j < 10; j++) {
                document.getElementById(i + '' + j).textContent = null       
            }
        }
    this.init()
    },
   async moveWest(){
       await query.execute(conn, 'proj_kr', 'DELETE { ?c a :CellPlayer. ?ball a :Ball } INSERT { ?x a :CellPlayer. ?newBall a :Ball. }WHERE{{ ?c a :CellPlayer. ?x a :IsWest. FILTER NOT EXISTS { ?x a :Ball .}} UNION { ?c a :CellPlayer. ?x a :IsWest. ?ball a :IsWest. ?ball a :Ball. ?ball :hasWest ?newBall. ?newBall a :NotWall. }}', 
                  'application/sparql-results+json', {
      reasoning: true
    })
    for (let i = 0; i < 10; i++) {
            for (let j = 0; j < 10; j++) {
                document.getElementById(i + '' + j).textContent = null       
            }
        }
    this.init()
    },
    }
  
}
const conn = new Connection({
      username: 'admin',
      password: 'admin',
      endpoint: 'http://localhost:5820',
    });
</script>
<style>
.container {
    margin-left: auto;
    margin-right: auto;
    width: 80%;
}
table {
    table-layout:fixed;
    width: 100%; 
}
table td {
    /* width: 30px; */
    height: 50px;
    overflow: hidden;
    text-overflow: ellipsis;
    border: 2px solid;
}
#up {
  width: 25%;
  margin-left: auto;
  margin-right: auto;
}
#down {
  width: 25%;
  margin-left: auto;
  margin-right: auto;
}
#right {
  width: 25%;
  margin-right: auto;
}
#left {
  width: 25%;
  margin-left: auto;
}
</style>