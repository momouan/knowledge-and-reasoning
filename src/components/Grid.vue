<template>
  <div class="container">
    <div id="home">
      <p> 21/22 - Knowledge and Reasoning Project</p>
      <h3>A Good Snowman Is Hard To Build</h3>
      <hr />
      <br />
    </div>
    <table id="controls">
      <tr>
        <td></td>
        <td><button id="start" @click="re_init()">(RE) START</button></td>
        <td></td>
      </tr>
      <tr>
        <td></td>
        <td><button id="up" @click="move_or_not('North')">Up</button></td>
        <td></td>
      </tr>
      <tr>
        <td><button id="left" @click="move_or_not('West')">Left</button></td>
        <td></td>
        <td><button id="right" @click="move_or_not('East')">Right</button></td>
      </tr>
      <tr>
        <td></td>
        <td><button id="down" @click="move_or_not('South')">Down</button></td>
        <td></td>
      </tr>
    </table>
    <table id="game_grid">
      <tr v-for="(i, ind1) in 10" :key="`i-${ind1}`" :id="`${i - 1}`">
        <td
          v-for="(j, ind2) in 10"
          :key="`j-${ind2}`"
          :id="`${i - 1}${ind2}`"
        ></td>
      </tr>
    </table>
  </div>
</template>
<script>
const { Connection, query } = require("stardog");
export default {
  name: "Grid",
  data() {
    return {
      ball_type: "",
      combination: [],
      next_cell_type: "",
      cap_move: "",
      combination_cell: "",
      player: "",
      small: "",
      medium: "",
      big: "",
      player_img: require("../assets/images/Player.png"),
      small_img: require("../assets/images/Small.png"),
      medium_img: require("../assets/images/Medium.png"),
      big_img: require("../assets/images/Big.png"),
      small_medium_img: require("../assets/images/SmallMedium.png"),
      small_big_img: require("../assets/images/SmallBig.png"),
      medium_big_img: require("../assets/images/MediumBig.png"),
      small_medium_big_img: require("../assets/images/SmallMediumBig.png"),
    };
  },
  props: {},
  computed: {},
  methods: {
    async init() {
      this.clear_grid();
      await this.get_player();
      if (this.combination.length == 3) {
        await this.get_combination();
      } 
      else if (this.combination.length == 2) {
        await this.get_combination();
        if (this.combination.includes("Small") && this.combination.includes("Medium"))
          this.get_big();
        else if (this.combination.includes("Medium") && this.combination.includes("Big"))
          this.get_small();
        else if (this.combination.includes("Small") && this.combination.includes("Big"))
          this.get_medium();
      } 
      else 
        await this.get_balls();
    },
    async update_grid() {
      this.clear_grid();
      await this.init();
    },
    clear_grid() {
      for (let i = 0; i < 10; i++) {
        for (let j = 0; j < 10; j++) {
          document.getElementById(i + "" + j).innerHTML = "";
        }
      }
    },
    async re_init() {
      await query.execute(
        conn,
        "proj_kr",
        `DELETE {?a a :CellPlayer; a :Small; a :Medium; a :Big; a :SmallMedium; a :SmallBig; a :MediumBig; a :SmallMediumBig; a :Combination}
         WHERE  { ?a a :Cell. }`,
        "application/sparql-results+json",
        {
          reasoning: true,
        }
      );
      await query.execute(
        conn,
        "proj_kr",
        `INSERT { :Cell01 a :CellPlayer. :Cell16 a :Small. :Cell33 a :Medium. :Cell25 a :Big}
         WHERE  { :Cell01 a :Cell. :Cell16 a :Cell. :Cell33 a :Cell. :Cell25 a :Cell. }`,
        "application/sparql-results+json",
        {
          reasoning: true,
        }
      );
      this.combination = [];
      await this.update_grid();
    },
    async move(dir) {
      await query.execute(
        conn,
        "proj_kr",
        `DELETE { ?c a :CellPlayer. ?ball a :` + this.ball_type + ` } 
         INSERT { ?x a :CellPlayer. ?newBall a :` + this.ball_type +`. }
         WHERE{{ ?c a :CellPlayer. ?x a :Is` + dir +`. 
            FILTER NOT EXISTS { ?x a :` + this.ball_type + `.}} 
            UNION { ?c a :CellPlayer. 
                    ?x a :Is` + dir + `. 
                    ?ball a :Is` + dir + `. 
                    ?ball a :` + this.ball_type + `. 
                    ?ball :has` + dir + ` ?newBall. 
                    ?newBall a :NotWall. }}`,
        "application/sparql-results+json",
        {
          reasoning: true,
        }
      );
      await this.check_combination(dir);
      await this.update_grid();
      if (this.combination.length == 3) {
        alert("game completed, click 'OK' to play again!");
        this.re_init();
      }
    },
    async move_or_not(dir) {
      await this.is_combination(dir);
      if (this.cap_move) {
        await this.get_ball_type(dir);
        await this.isnext_cell_type(dir);
        if (
          (this.ball_type == "Big" && this.next_cell_type != "na") ||
          (this.ball_type == "Medium" && this.next_cell_type == "Small") ||
          this.next_cell_type == "SmallBig" ||
          this.next_cell_type == "SmallMedium"
        )
          this.dont_move();
        else this.move(dir);
      } else {
        this.dont_move();
      }
    },
    async isnext_cell_type(dir) {
      await query
        .execute(
          conn,
          "proj_kr",
          `SELECT ?type
           WHERE { ?c a :Is` + dir + `.
                   ?c :has`+ dir +` ?x.  
                   ?x a ?type.  
                  {?type rdfs:subClassOf :Ball} 
                  UNION {?type rdfs:subClassOf :Combination} 
                  MINUS{ VALUES (?type) { (:Ball) (:Combination)}}}`,
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        )
        .then(({ body }) => {
          if (body.results.bindings.length > 0) {
            this.next_cell_type = body.results.bindings[0].type.value.replace(
              "http://www.semanticweb.org/djam/ontologies/2021/9/snowman#",
              ""
            );
          } else this.next_cell_type = "na";
        });
    },
    async get_player() {
      await query
        .execute(
          conn,
          "proj_kr",
          "SELECT ?c WHERE { ?c a :CellPlayer. }",
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        )
        .then(({ body }) => {
          this.player = body.results.bindings[0].c.value.replace(
            "http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell",
            ""
          );
          var img = document.createElement("img");
          img.src = this.player_img;
          img.id = "player";
          img.alt = "Player";
          document.getElementById(this.player).appendChild(img);
        });
    },
    async get_balls() {
      this.get_small();
      this.get_medium();
      this.get_big();
    },
    async get_ball_type(dir) {
      await query
        .execute(
          conn,
          "proj_kr",
          `SELECT ?type 
           WHERE { ?c a :Is` + dir + `. 
                   ?c a ?type. 
                   ?type rdfs:subClassOf :Ball. MINUS{ VALUES (?type) { (:Ball)}}}`,
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        )
        .then(({ body }) => {
          if (body.results.bindings.length != 0)
            this.ball_type = body.results.bindings[0].type.value.replace(
              "http://www.semanticweb.org/djam/ontologies/2021/9/snowman#",
              ""
            );
          else this.ball_type = "na";
        });
    },
    get_small() {
      query
        .execute(
          conn,
          "proj_kr",
          "SELECT ?c WHERE { ?c a :Small. }",
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        )
        .then(({ body }) => {
          this.small = body.results.bindings[0].c.value.replace(
            "http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell",
            ""
          );
          var img = document.createElement("img");
          img.src = this.small_img;
          img.id = "small";
          img.alt = "Small";
          document.getElementById(this.small).appendChild(img);
        });
    },
    get_medium() {
      query
        .execute(
          conn,
          "proj_kr",
          "SELECT ?c WHERE { ?c a :Medium. }",
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        )
        .then(({ body }) => {
          this.medium = body.results.bindings[0].c.value.replace(
            "http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell",
            ""
          );
          var img = document.createElement("img");
          img.src = this.medium_img;
          img.id = "medium";
          img.alt = "Medium";
          document.getElementById(this.medium).appendChild(img);
        });
    },
    get_big() {
      query
        .execute(
          conn,
          "proj_kr",
          "SELECT ?c WHERE { ?c a :Big. }",
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        )
        .then(({ body }) => {
          this.big = body.results.bindings[0].c.value.replace(
            "http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell",
            ""
          );
          var img = document.createElement("img");
          img.src = this.big_img;
          img.id = "big";
          img.alt = "Big";
          document.getElementById(this.big).appendChild(img);
        });
    },
    async check_combination(dir) {
      if (this.combination.length == 0) {
        await query
          .execute(
            conn,
            "proj_kr",
            `SELECT ?type 
            WHERE { ?c a :Is` + dir +`. ?c a ?type. 
                    ?type rdfs:subClassOf :Ball. MINUS{ VALUES (?type) { (:Ball) }}}`,
            "application/sparql-results+json",
            {
              reasoning: true,
            }
          )
          .then(({ body }) => {
            if (body.results.bindings.length > 1) {
              body.results.bindings.forEach((element) => {
                this.combination.push(
                  element.type.value.replace(
                    "http://www.semanticweb.org/djam/ontologies/2021/9/snowman#",
                    ""
                  )
                );
              });
              this.combination.sort();
              this.combination.reverse();
              this.make_combination(dir);
            }
          });
      } else if (this.combination.length == 2) {
        await query
          .execute(
            conn,
            "proj_kr",
            `SELECT ?type
             WHERE { ?c a :Is` + dir + `. ?c a :Combination. ?c a :Ball.
                     ?c a ?type.
                     ?type rdfs:subClassOf :Ball. MINUS{ VALUES (?type) { (:Ball) }}}`,
            "application/sparql-results+json",
            {
              reasoning: true,
            }
          )
          .then(({ body }) => {
            if (body.results.bindings.length != 0) {
              this.combination.push(
                body.results.bindings[0].type.value.replace(
                  "http://www.semanticweb.org/djam/ontologies/2021/9/snowman#",
                  ""
                )
              );
              this.combination.sort();
              this.combination.reverse();
              this.make_combination(dir);
            }
          });
      }
    },
    async make_combination(dir) {
      if (this.combination.length == 2) {
        await query.execute(
          conn,
          "proj_kr",
          `DELETE { ?c a :Ball, :` + this.combination[0] + `, :` + this.combination[1] + `.} 
           INSERT { ?c a :Combination. ?c a :` + this.combination[0] + this.combination[1] + `.}
           WHERE  { ?c a :Is` + dir + `. }`,
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        );
      } else if (this.combination.length == 3) {
        await query.execute(
          conn,
          "proj_kr",
          `DELETE { ?c a :Ball, :` + this.combination[0] + `, :` + this.combination[1] + `, :` + this.combination[2] + `.} 
           INSERT { ?c a :Combination. ?c a :` + this.combination[0] + this.combination[1] + this.combination[2] + `.}
           WHERE  { ?c a :Is` + dir + `. }`,
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        );
      }
    },
    async get_combination() {
      await query
        .execute(
          conn,
          "proj_kr",
          "SELECT ?c WHERE { ?c a :Combination. }",
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        )
        .then(({ body }) => {
          if (body.results.bindings.length == 0) 
            this.update_grid();
          this.combination_cell = body.results.bindings[0].c.value.replace(
            "http://www.semanticweb.org/djam/ontologies/2021/9/snowman#Cell",
            ""
          );
          var img = document.createElement("img");
          if (this.combination.length == 2) {
            var str = "" + this.combination[0] + this.combination[1] + "";
            switch (str) {
              case "SmallMedium":
                img.src = this.small_medium_img;
                img.alt = "SmallMedium";
                document.getElementById(this.combination_cell).appendChild(img);
                break;
              case "SmallBig":
                img.src = this.small_big_img;
                img.alt = "SmallBig";
                document.getElementById(this.combination_cell).appendChild(img);
                break;
              case "MediumBig":
                img.src = this.medium_big_img;
                img.alt = "MediumBig";
                document.getElementById(this.combination_cell).appendChild(img);
                break;
            }
          } else if (this.combination.length == 3) {
            img.src = this.small_medium_big_img;
            img.alt = "SmallMediumBig";
            document.getElementById(this.combination_cell).appendChild(img);
          }
        });
    },
    async dont_move() {},
    async is_combination(dir) {
      await query
        .execute(
          conn,
          "proj_kr",
          `SELECT ?c WHERE { ?c a :Is` + dir + `. ?c a :Combination. }`,
          "application/sparql-results+json",
          {
            reasoning: true,
          }
        )
        .then(({ body }) => {
          if (body.results.bindings.length > 0) this.cap_move = false;
          else this.cap_move = true;
        });
    },
  },
};
const conn = new Connection({
  username: "admin",
  password: "admin",
  endpoint: "http://localhost:5820",
});
</script>
<style>
.container {
  width: 80%;
  margin-left: auto;
  margin-right: auto;
}
#game_grid {
  table-layout: fixed;
  width: 80%;
  margin-left: auto;
  float: right;
}
#game_grid td {
  height: 50px;
  overflow: hidden;
  text-overflow: ellipsis;
  border: 2px solid;
}
#controls {
  float: left;
  margin-top: 300px;
}
#small {
  width: 40px;
  height: 20px;
}
#medium {
  width: 60px;
  height: 40px;
}
#big {
  width: 80px;
  height: 45px;
}
#start {
  margin-bottom: 100px;
}
p {
  font-size: 12px;
}
</style>