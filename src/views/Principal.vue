<template>
  <v-container fluid grid-list-lg>
    <v-layout row wrap justify-center>
      <v-flex xs12>
        <v-alert
          :type="type"
          :value="text != ''"
          dismissible
          @click="text = ''"
        >
          {{ text }}
        </v-alert>
        <v-card v-if="token == ''">
          <v-toolbar :color="panelHeaderColor" dark>
            <v-toolbar-title>Por favor autentique-se primeiro:</v-toolbar-title>
          </v-toolbar>
          <v-card-text>
            <v-form ref="form" lazy-validation>
              <v-text-field
                prepend-icon="email"
                name="email"
                v-model="form.email"
                label="Email"
                type="email"
                :rules="regraEmail"
                required
              />
              <v-text-field
                id="password"
                prepend-icon="lock"
                name="password"
                v-model="form.password"
                label="Password"
                type="password"
                :rules="regraPassword"
                required
              />
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" type="submit" @click="loginUtilizador"
              >Login</v-btn
            >
          </v-card-actions>
        </v-card>
      </v-flex>
    </v-layout>

    <v-layout row wrap justify-center>
      <v-flex xs12>
        <v-card v-if="token != ''">
          <v-toolbar :color="panelHeaderColor" dark>
            <v-toolbar-title>Seleção de parâmetros</v-toolbar-title>
          </v-toolbar>
          <v-card-text>
            <v-layout wrap v-if="listaProcReady">
              <v-flex xs2>
                <v-subheader>Processo:</v-subheader>
              </v-flex>
              <v-flex xs10>
                <v-select
                  item-text="label"
                  item-value="value"
                  v-model="processo"
                  :items="listaProcessos"
                  label="Selecione o processo de partida"
                  solo
                  dense
                />
              </v-flex>
            </v-layout>
            <v-layout v-else>
              <p>A carregar os processos...</p>
            </v-layout>

            <v-layout>
              <v-flex xs2>
                <v-subheader>Niveis de profundidade:</v-subheader>
              </v-flex>
              <v-flex xs10>
                <v-select
                  v-model="niveis"
                  :items="listaNiveis"
                  label="Selecione o nível de profundidade pretendido"
                  solo
                  dense
                />
              </v-flex>
            </v-layout>

            <v-layout>
              <v-flex>
                <v-btn
                  round
                  color="info"
                  @click="
                    calcRel(processo);
                    calculoTotal = false;
                  "
                >
                  Calcular
                </v-btn>
              </v-flex>
            </v-layout>

            <v-layout>
              <v-flex>
                <v-btn
                  round
                  color="info"
                  @click="
                    calcRelTotal();
                    calculoTotal = true;
                  "
                >
                  Calcular todas as travessias de todos os processos
                </v-btn>
              </v-flex>
            </v-layout>

            <v-layout v-if="aCalcular">
              <v-flex>
                <v-btn round color="error" @click="cancelar()"
                  >Cancelar Calculo</v-btn
                >
              </v-flex>
            </v-layout>

            <v-layout v-if="fechoTotalCalculado">
              <v-flex>
                <v-btn round color="info" @click="writeJSON()">
                  Enviar para API de dados
                </v-btn>
              </v-flex>
            </v-layout>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>

    <v-layout row wrap justify-center v-if="token != ''">
      <v-flex xs12>
        <v-card v-if="calculoTotal">
          <v-toolbar :color="panelHeaderColor" dark>
            <v-toolbar-title>Todos os Resultados</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-toolbar-items class="hidden-sm-and-down">
              <v-btn flat>Nível de profundidade máxima</v-btn>
            </v-toolbar-items>
          </v-toolbar>
          <v-card-text>
            <v-layout v-if="!fechoTotalCalculado">
              <v-flex xs12>
                <p>A calcular a travessia... Código: {{ processo }}</p>
              </v-flex>
            </v-layout>
            <v-layout wrap>
              <v-flex xs12>
                {{ resultadosJSON }}
              </v-flex>
            </v-layout>
          </v-card-text>
        </v-card>
        <v-card v-else>
          <v-toolbar :color="panelHeaderColor" dark>
            <v-toolbar-title>Resultados</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-toolbar-items class="hidden-sm-and-down">
              <v-btn flat>Nível de profundidade: {{ profundidade }}</v-btn>
            </v-toolbar-items>
          </v-toolbar>
          <v-card-text>
            <v-layout wrap v-if="listaResultados.length > 0">
              <v-flex xs12>
                <v-data-table
                  :headers="resultadosHeaders"
                  :items="listaResultados"
                  class="elevation-1"
                >
                  <template v-slot:headers="props">
                    <tr>
                      <th
                        v-for="h in props.headers"
                        :key="h.value"
                        class="body-2 font-weight-bold"
                      >
                        {{ h.text }}
                      </th>
                    </tr>
                  </template>

                  <template v-slot:items="props">
                    <tr>
                      <td>{{ props.item.codigo }}</td>
                      <td>{{ props.item.titulo }}</td>
                      <td>{{ props.item.relacao }}</td>
                      <td>{{ props.item.nivel }}</td>
                    </tr>
                  </template>
                </v-data-table>
              </v-flex>
            </v-layout>
            <v-layout v-else>
              <p>A calcular a travessia...</p>
            </v-layout>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
const lhost = require("@/config/global").host;
const axios = require("axios");

export default {
  data() {
    return {
      text: "",
      token: "",
      type: "success",
      form: {
        email: "",
        password: ""
      },
      regraEmail: [
        v => !!v || "Email é obrigatório.",
        v => /.+@.+/.test(v) || "Email tem de ser válido."
      ],
      regraPassword: [v => !!v || "Password é obrigatório."],
      processo: "",
      listaProcessos: [],
      listaProcReady: false,
      listaResultados: [],
      resultadosReady: false,
      profundidade: 0,
      panelHeaderColor: "indigo accent-4",
      resultadosHeaders: [
        { text: "Código", align: "left", value: "codigo" },
        { text: "Título", value: "titulo" },
        { text: "Relação", value: "relacao" },
        { text: "Distância", value: "nivel" }
      ],
      niveis: 0,
      listaNiveis: [
        { text: "1 - Vizinhos imediatos", value: 1 },
        { text: "2 - Vizinhos a 2 relações de distância", value: 2 },
        { text: "3 - Vizinhos a 3 relações de distância", value: 3 },
        { text: "4 - Vizinhos a 4 relações de distância", value: 4 },
        { text: "5 - Vizinhos a 5 relações de distância", value: 5 },
        { text: "6 - Vizinhos a 6 relações de distância", value: 6 },
        { text: "7 - Vizinhos a 7 relações de distância", value: 7 },
        { text: "8 - Vizinhos a 8 relações de distância", value: 8 },
        { text: "9 - Vizinhos a 9 relações de distância", value: 9 },
        { text: "10 - Vizinhos a 10 relações de distância", value: 10 },
        { text: "Todos - irá tentar calcular o alcance total", value: 1000 }
      ],
      stackProc: [],
      visitados: [],
      calculoTotal: false,
      fechoTotalCalculado: false,
      resultadosJSON: [],
      aCalcular: false,
      travessias: []
    };
  },

  methods: {
    loadProcessos: async function() {
      try {
        var response = await axios.get(lhost + "/api/classes?nivel=3", {
          headers: {
            Authorization: "token " + this.token
          }
        });
        this.listaProcessos = response.data
          .map(function(item) {
            return {
              label: item.codigo + " - " + item.titulo,
              value: item.id.split("#c")[1]
            };
          })
          .sort(function(a, b) {
            return a.label.localeCompare(b.label);
          });
        this.listaProcReady = true;
      } catch (erro) {
        this.text = erro.response.data;
        this.type = "error";
      }
    },

    loadRels: async function(p, profundidade, relacao) {
      try {
        var response = await axios.get(
          lhost + "/api/classes/c" + p + "/procRel/" + relacao,
          {
            headers: {
              Authorization: "token " + this.token
            }
          }
        );
        return response.data.map(function(item) {
          return {
            codigo: item.codigo,
            titulo: item.titulo,
            relacao: relacao,
            nivel: profundidade
          };
        });
      } catch (erro) {
        throw erro.response.data;
      }
    },

    calcRel: async function(processo) {
      try {
        this.fechoTotalCalculado = false;
        this.text = "";
        this.listaResultados = [];
        this.stackProc = [];
        this.visitados = [];
        this.stackProc.push({ listaProc: [], nivel: 1 });
        this.stackProc[0].listaProc.push(processo);
        this.visitados.push(processo); // Processo inicial está no índice 0
        this.profundidade = 1;
        var p;

        while (
          this.profundidade <= this.niveis &&
          this.stackProc[this.profundidade - 1].listaProc.length > 0
        ) {
          this.stackProc.push({ listaProc: [], nivel: this.profundidade + 1 });
          for (
            var i = 0;
            i < this.stackProc[this.profundidade - 1].listaProc.length;
            i++
          ) {
            p = this.stackProc[this.profundidade - 1].listaProc[i];

            if (p < processo && this.calculoTotal) {
              this.listaResultados = await this.juntaNovos(
                this.listaResultados,
                this.travessias[p]
              );
            } else {
              var comp = await this.loadRels(
                p,
                this.profundidade,
                "eComplementarDe"
              );
              if (comp.length > 0) {
                this.listaResultados = await this.juntaNovos(
                  this.listaResultados,
                  comp
                );
                this.stackProc[this.profundidade].listaProc = this.stackProc[
                  this.profundidade
                ].listaProc.concat(
                  await this.juntaNovosVisitas(
                    this.visitados,
                    this.filtra(comp)
                  )
                );
              }

              var sint = await this.loadRels(
                p,
                this.profundidade,
                "eSintetizadoPor"
              );
              if (sint.length > 0) {
                this.listaResultados = await this.juntaNovos(
                  this.listaResultados,
                  sint
                );
                this.stackProc[this.profundidade].listaProc = this.stackProc[
                  this.profundidade
                ].listaProc.concat(
                  await this.juntaNovosVisitas(
                    this.visitados,
                    this.filtra(sint)
                  )
                );
              }

              var sup = await this.loadRels(
                p,
                this.profundidade,
                "eSuplementoPara"
              );
              if (sup.length > 0) {
                this.listaResultados = await this.juntaNovos(
                  this.listaResultados,
                  sup
                );
                this.stackProc[this.profundidade].listaProc = this.stackProc[
                  this.profundidade
                ].listaProc.concat(
                  await this.juntaNovosVisitas(this.visitados, this.filtra(sup))
                );
              }
            }
          }
          this.profundidade++;
        }

        this.listaResultados.sort(function(a, b) {
          return a.codigo.localeCompare(b.codigo);
        });

        this.resultadosReady = true;
        if (!this.calculoTotal) {
          this.text =
            "Cálculo terminado: " +
            this.listaResultados.length +
            " processos. Pode realizar outro cálculo.";
          this.type = "success";
        }
        return this.listaResultados;
      } catch (erro) {
        this.text = erro;
        this.type = "error";
      }
    },

    filtra: function(lproc) {
      return lproc.map(function(p) {
        return p.codigo;
      });
    },

    juntaNovosVisitas: async function(visitados, candidatos) {
      var res = [];
      for (var i = 0; i < candidatos.length; i++) {
        var index = visitados.indexOf(candidatos[i]);
        if (index == -1) {
          visitados.push(candidatos[i]);
          res.push(candidatos[i]);
        }
      }
      return res;
    },

    juntaNovos: async function(existentes, candidatos) {
      for (var i = 0; i < candidatos.length; i++) {
        if (existentes.length > 0) {
          var index = existentes.findIndex(
            p => p.codigo == candidatos[i].codigo
          );
          if (index == -1) {
            existentes.push(candidatos[i]);
          }
        } else {
          existentes.push(candidatos[i]);
        }
      }
      return existentes;
    },

    calcRelTotal: async function() {
      this.niveis = 1000;
      this.aCalcular = true;
      this.fechoTotalCalculado = false;
      this.text = "";

      for (var i = 0; i < this.listaProcessos.length; i++) {
        this.processo = this.listaProcessos[i].value;
        var resultado = await this.calcRel(this.listaProcessos[i].value);

        var lista = [];
        for (var j = 0; j < resultado.length; j++) {
          lista.push(resultado[j].codigo);
        }

        this.travessias[this.processo] = resultado;
        this.resultadosJSON.push({
          processo: this.processo,
          travessia: lista
        });
      }
      this.fechoTotalCalculado = true;
      this.aCalcular = false;
      this.text = "Cálculo total terminado: Pode realizar outro cálculo.";
      this.type = "success";
    },

    writeJSON: async function() {
      try {
        var response = await axios.post(
          lhost + "/api/travessia",
          this.resultadosJSON,
          {
            headers: {
              Authorization: "token " + this.token
            }
          }
        );
        this.text = "Enviado para a API de dados";
        this.type = "success";
      } catch (e) {
        this.text = e.response.data;
        this.type = "error";
      }
    },

    cancelar: function() {
      window.location.reload();
    },

    loginUtilizador() {
      if (this.$refs.form.validate()) {
        axios
          .post(lhost + "/api/users/login", {
            username: this.$data.form.email,
            password: this.$data.form.password
          })
          .then(res => {
            if (res.data.token != undefined) {
              this.text = "Login efetuado com sucesso!";
              this.type = "success";
              this.token = res.data.token;
              this.loadProcessos();
            } else {
              this.text =
                "Ocorreu um erro ao realizar o login: Por favor verifique as suas credenciais!";
              this.type = "error";
            }
          })
          .catch(err => {
            this.text =
              "Ocorreu um erro ao realizar o login: Por favor verifique as suas credenciais!";
            this.type = "error";
          });
      } else {
        this.text = "Por favor preencha todos os campos!";
        this.type = "error";
      }
    }
  }
};
</script>
