<template>
    <div class="container">
      <button @click="init()">INIT THE SHIT</button>
      <div>
      <button id="up" @click="move('North')">Up</button>
      <button id="down" @click="move('South')">Down</button>
      <button id="right" @click="move('East')">Right</button>
      <button id="left" @click="move('West')">Left </button>
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
export default {
  name: 'Grid',
  data(){
      return {
        ball_type: "Small",
        north: "",
        south: "",
        est: "",
        west: "",
      }
  },
  props: {
    
  },
  computed: {

  },
  methods: {
     async init(){
       this.get_player()
       this.get_balls()
       //await this.get_surroundings()
    
   },
   async re_init(){
     for (let i = 0; i < 10; i++) {
            for (let j = 0; j < 10; j++) {
                document.getElementById(i + '' + j).textContent = null       
            }
        }
    this.init()
   },
   async move(dir){
    await this.get_ball_type(dir)
       await query.execute(conn, 'proj_kr', `DELETE { ?c a :CellPlayer. ?ball a :`+ this.ball_type +` } 
                                             INSERT { ?x a :CellPlayer. ?newBall a :`+ this.ball_type +`. }
                                             WHERE{{ ?c a :CellPlayer. ?x a :Is`+dir+`. 
                                                      FILTER NOT EXISTS { ?x a :`+ this.ball_type +`.}} 
                                                      UNION { ?c a :CellPlayer. 
                                                              ?x a :Is`+dir+`. 
                                                              ?ball a :Is`+dir+`. 
                                                              ?ball a :`+ this.ball_type +`. 
                                                              ?ball :has`+dir+` ?newBall. 
                                                              ?newBall a :NotWall. }}`, 
                  'application/sparql-results+json', {
      reasoning: true
    })
    this.re_init()
    },
    // async get_surroundings(){
    //   await query.execute(conn, 'proj_kr', `SELECT ?north ?south ?west ?east 
    //                                         WHERE {{?c a :IsEast. ?c rdf:type ?east.
    //                                                 ?east rdfs:subClassOf :Ball. MINUS{ VALUES (?east) { (:Ball)} }}
    //                                         UNION { ?c a :IsWest. ?c rdf:type ?west.
    //                                                 ?west rdfs:subClassOf :Ball. MINUS{ VALUES (?west) { (:Ball)} }}
    //                                         UNION { ?c a :IsNorth. ?c rdf:type ?north. 
    //                                                 ?north rdfs:subClassOf :Ball. MINUS{ VALUES (?north) { (:Ball)} }}
    //                                         UNION { ?c a :IsSouth. ?c rdf:type ?south.
    //                                                 ?south rdfs:subClassOf :Ball. MINUS{ VALUES (?south) { (:Ball)} }}}`,
    //                 'application/sparql-results+json', {
    //    reasoning: true
    // }).then(({ body }) => { 
    //   console.log(body.results.bindings)
    //   this.north = this.south = this.est = this.west = "na"
    //   if(body.results.bindings.length != 0){
    //     console.log(body.results.bindings[0][0])
    //     console.log("zzeb")
    //     }
    // });
    // },
    get_player(){
        query.execute(conn, 'proj_kr', 'SELECT ?c WHERE { ?c a :CellPlayer. }', 
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => { 
        body = body.results.bindings[0].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(body).textContent = "Player"
      });
      },
    get_balls(){
        query.execute(conn, 'proj_kr', 'SELECT ?c WHERE {{ ?c a :Small. } UNION { ?c a :Medium } UNION { ?c a :Big }}',
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => { 
        console.log(body.results.bindings)
        this.small = body.results.bindings[0].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.small).textContent = "Small";
        this.medium = body.results.bindings[1].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.medium).textContent = "Medium";
        this.big = body.results.bindings[2].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.big).textContent = "Big";
      });
    },
    async get_ball_type(dir){
      await query.execute(conn, 'proj_kr', `SELECT ?type WHERE {?c a :Is`+dir+`. ?c rdf:type ?type. ?type rdfs:subClassOf :Ball. MINUS{ VALUES (?type) { (:Ball)}}}`, 'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => {
        if(body.results.bindings.length != 0){
          this.ball_type = body.results.bindings[0].type.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#", "");
        }
      });
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