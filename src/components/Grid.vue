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
        <td v-for="(j, ind2) in 10" :key="`j-${ind2}`" :id="`${i-1}${ind2}`"></td>
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
        ball_type: "",
        combination: ["Small", "Big"],
        cap_move: "",
        combination_cell: "",
        player: "",
        small: "",
        medium: "",
        big: "",
      }
  },
  props: {
    
  },
  computed: {
  
  },
  methods: {
     async init(){
        await this.get_player()
        if(this.combination.length == 3){
          await this.get_combination();
        }
        else if(this.combination.length == 2){
          await this.get_combination();
          if(this.combination.includes("Small") && this.combination.includes("Medium"))
            this.get_big();
          else if(this.combination.includes("Medium") && this.combination.includes("Big"))
            this.get_small();
          else if(this.combination.includes("Small") && this.combination.includes("Big"))
            this.get_medium();
        }
        else
          await this.get_balls()
   },
   async re_init(){
     for (let i = 0; i < 10; i++) {
            for (let j = 0; j < 10; j++) {
                document.getElementById(i + '' + j).textContent = null       
            }
        }
    await this.init()
   },
   async move(dir){
     await this.is_combination(dir)
     if(this.cap_move){
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
      });
      await this.check_combination(dir)
      await this.re_init()
      }
      else
        this.dont_move()
    },
    async get_player(){
        await query.execute(conn, 'proj_kr', 'SELECT ?c WHERE { ?c a :CellPlayer. }', 
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => { 
        this.player = body.results.bindings[0].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.player).textContent = "Player"
      });
      },
    async get_balls(){
        await query.execute(conn, 'proj_kr', 'SELECT ?small ?medium ?big WHERE {{ ?small a :Small. } UNION { ?medium a :Medium } UNION { ?big a :Big }}',
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => {
        this.small = body.results.bindings[0].small.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.small).textContent = "Small";
        this.medium = body.results.bindings[1].medium.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.medium).textContent = "Medium";
        this.big = body.results.bindings[2].big.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.big).textContent = "Big";
      });
    },
    async get_ball_type(dir){ 
      await query.execute(conn, 'proj_kr', `SELECT ?type 
                                            WHERE { ?c a :Is`+dir+`. 
                                                    ?c a ?type. 
                                                    ?type rdfs:subClassOf :Ball. MINUS{ VALUES (?type) { (:Ball)}}}`, 
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => {
        if(body.results.bindings.length != 0)
          this.ball_type = body.results.bindings[0].type.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#", "");
        else
          this.ball_type = "na"
      });
    },
    get_small(){
      query.execute(conn, 'proj_kr', 'SELECT ?c WHERE { ?c a :Small. }',
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => {
        this.small = body.results.bindings[0].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.small).textContent = "Small";
      });
    },
    get_medium(){
      query.execute(conn, 'proj_kr', 'SELECT ?c WHERE { ?c a :Medium. }',
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => {
        this.medium = body.results.bindings[0].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.medium).textContent = "Medium";
      });
    },
    get_big(){
      query.execute(conn, 'proj_kr', 'SELECT ?c WHERE { ?c a :Big. }',
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => {
        this.big = body.results.bindings[0].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        document.getElementById(this.big).textContent = "Big";
      });
    },
    async check_combination(dir){
      if(this.combination.length == 0){
        await query.execute(conn, 'proj_kr', `SELECT ?type 
                                        WHERE { ?c a :Is`+dir+`. ?c a ?type. 
                                                ?type rdfs:subClassOf :Ball. MINUS{ VALUES (?type) { (:Ball) }}}`,
                        'application/sparql-results+json', {
          reasoning: true
          }).then(({ body }) => {
            if(body.results.bindings.length > 1){
              body.results.bindings.forEach(element => {
                this.combination.push(element.type.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#", ""));
              });
              this.combination.sort();
              this.combination.reverse();
              this.sleep(200);
              this.make_combination(dir);
          }
        });
      }
      else if(this.combination.length == 2){
        await query.execute(conn, 'proj_kr', `SELECT ?type
                                              WHERE {
                                                  ?c a :Is`+dir+`. ?c a :Combination. ?c a :Ball.
                                                  ?c a ?type.
                                                  ?type rdfs:subClassOf :Ball. MINUS{ VALUES (?type) { (:Ball) }}}`,
                          'application/sparql-results+json', {
       reasoning: true
       }).then(({ body }) => {
         if(body.results.bindings.length != 0){
          this.combination.push(body.results.bindings[0].type.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#", ""));
          this.combination.sort();
          this.combination.reverse();
          this.sleep(200);
          this.make_combination(dir);
         }
       });
      }
    },
    async make_combination(dir){
          if(this.combination.length == 2){
              await query.execute(conn, 'proj_kr', `DELETE { ?c a :Ball, :`+this.combination[0]+`, :`+this.combination[1]+`.} 
                                              INSERT { ?c a :`+this.combination[0]+this.combination[1]+`.}
                                              WHERE  { ?c a :Is`+dir+`. }`,
                       'application/sparql-results+json', {
                 reasoning: true        
                       });
            }
            else if(this.combination.length == 3){
              await query.execute(conn, 'proj_kr', `DELETE { ?c a :Ball, :`+this.combination[0]+`, :`+this.combination[1]+`, :`+this.combination[2]+`.} 
                                              INSERT { ?c a :`+this.combination[0]+this.combination[1]+this.combination[2]+`.}
                                              WHERE  { ?c a :Is`+dir+`. }`,
                       'application/sparql-results+json', {
                 reasoning: true        
                       });
            }
      },
    async get_combination(){
      await query.execute(conn, 'proj_kr', 'SELECT ?c WHERE { ?c a :Combination. }',
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => {
        this.combination_cell = body.results.bindings[0].c.value.replace("http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell", "");
        if(this.combination.length == 2)
          document.getElementById(this.combination_cell).textContent = ""+this.combination[0]+this.combination[1]+"";
        else if(this.combination.length == 3)
          document.getElementById(this.combination_cell).textContent = "SMB";
      });
    },
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    async dont_move(){  
    },
    async is_combination(dir){
      await query.execute(conn, 'proj_kr', `SELECT ?c WHERE { ?c a :Is`+dir+`. ?c a :Combination. }`,
                    'application/sparql-results+json', {
        reasoning: true
      }).then(({ body }) => {
        if(body.results.bindings.length > 0)
          this.cap_move = false 
        else
          this.cap_move = true
      });
    },
   }, 
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
    margin-left: auto;
    margin-right: auto;
    table-layout:fixed;
    width: 80%; 
    height: 80%;
}
table td {
    
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