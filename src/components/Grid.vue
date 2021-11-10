<template>
    <!-- <div v-for="(g, index) in grid" :key="`g-${index}`">
        <div v-for="(cell, index) in g.cells" :key="`cell-${index}`">
            <h4> i am inside </h4>
        </div>
    </div> -->
    <div class="container">
      <button @click="init()">INIT THE SHIT</button>
      <button @click="deplacerJoueur()">Turn left</button>
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
   deplacerJoueur(){
       query.execute(conn, 'proj_kr', 'delete { ?c a :CellPlayer. } insert { ?x a :CellPlayer} where { ?c a :CellPlayer. ?x a :IsWest. }', 
                  'application/sparql-results+json', {
      reasoning: true
    })
    for (let i = 0; i < 10; i++) {
            for (let j = 0; j < 10; j++) {
                console.log(document.getElementById(i + '' + j))          
                // console.log(i + '' + j)
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
    height: 80px;
    overflow: hidden;
    text-overflow: ellipsis;
    border: 2px solid;
}
</style>