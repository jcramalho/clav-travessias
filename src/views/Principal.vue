<template>
    <v-container fluid grid-list-lg>
        <v-layout row wrap justify-center>
            <v-flex xs12>
                <v-card>
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
                                    solo dense
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
                                    solo dense
                                />
                            </v-flex>
                        </v-layout>

                        <v-layout>
                            <v-flex>
                                <v-btn round color="info" @click="calcRel">Calcular</v-btn>
                            </v-flex>
                        </v-layout>

                    </v-card-text>
                </v-card>
            </v-flex>
        </v-layout>

        <v-layout row wrap justify-center>
            <v-flex xs12>
                <v-card>
                    <v-toolbar :color="panelHeaderColor" dark>
                        <v-toolbar-title>Resultados</v-toolbar-title>
                    </v-toolbar>
                    <v-card-text>
                        <v-layout wrap v-if="resultadosReady">
                            <v-flex xs12>
                                <v-data-table
                                    :headers="resultadosHeaders"
                                    :items="listaResultados"
                                    class="elevation-1"
                                >
                                    <template v-slot:headers="props">
                                        <tr>
                                            <th v-for="h in props.headers" :key="h.value" class="body-2 font-weight-bold">
                                                {{ h.text }}
                                            </th>
                                        </tr>
                                    </template>

                                    <template v-slot:items="props">
                                        <tr>
                                            <td>{{ props.item.codigo }}</td>
                                            <td> {{ props.item.titulo }} </td>
                                            <td> {{ props.item.relacao }} </td>
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
const lhost = require('@/config/global').host
const axios = require('axios')

export default {
    data () {
        return {
            processo: "",
            listaProcessos: [],
            listaProcReady: false,
            listaResultados: [],
            resultadosReady: false,
            panelHeaderColor: "indigo accent-4",
            resultadosHeaders: [
                { text: 'Código', align: 'left', value: 'codigo'},
                { text: 'Título', value: 'titulo' },
                { text: 'Relação', value: 'relacao' }
            ],
            niveis: 0,
            listaNiveis: [
                {text: "1 - Vizinhos imediatos", value: 1},
                {text: "2 - Vizinhos a 2 relações de distância", value: 2},
                {text: "3 - Vizinhos a 3 relações de distância", value: 3},
                {text: "5 - Vizinhos a 5 relações de distância", value: 5},
                {text: "Todos - irá tentar calcular o alcance total", value: 1000}
            ],
            stackProc: []
        }
    },

    methods: {
        // Carrega os potenciais pais da BD, quando alguém muda o nível para >1....................

        loadProcessos: async function () {
            try{
                var response = await axios.get(lhost + "/api/classes?nivel=3");
                this.listaProcessos = response.data.map(function (item) {
                    return {
                        label: item.codigo + " - " + item.titulo,
                        value: item.id.split('#c')[1],
                    }
                    }).sort(function (a, b) {
                        return a.label.localeCompare(b.label);
                    });
                this.listaProcReady = true;
            }
            catch(erro){
                console.log(erro);
            }
        },

        loadComplementares: async function(p){
            try{
                var response = await axios.get(lhost + "/api/classes/c" + p + "/procRel/eComplementarDe");
                return response.data.map(function (item) {
                    return {
                        codigo: item.codigo,
                        titulo: item.titulo,
                        relacao: "eComplementarDe"
                    }
                    });
            }
           catch(erro){
                console.log(erro);
            }
        },

        loadSintetizados: async function(p){
            try{
                var response = await axios.get(lhost + "/api/classes/c" + p + "/procRel/eSintetizadoPor");
                return response.data.map(function (item) {
                    return {
                        codigo: item.codigo,
                        titulo: item.titulo,
                        relacao: "eSintetizadoPor"
                    }
                });
            }
           catch(erro){
                console.log(erro);
            }
        },

        loadSuplementares: async function(p){
            try{
                var response = await axios.get(lhost + "/api/classes/c" + p + "/procRel/eSuplementoPara");
                return response.data.map(function (item) {
                    return {
                        codigo: item.codigo,
                        titulo: item.titulo,
                        relacao: "eSuplementoPara"
                    }
                });
            }
           catch(erro){
                console.log(erro);
            }
        },

        calcRel: async function(){
            try{
                this.stackProc = [];
                this.stackProc.push({listaProc: [], nivel: 1});
                this.stackProc[0].listaProc.push(this.processo);
                var profundidade = 1;
                var p;

                while(profundidade <= this.niveis){
                    this.stackProc.push({listaProc: [], nivel: profundidade+1});
                    for(var i=0; i < this.stackProc[profundidade-1].listaProc.length; i++){
                        p = this.stackProc[profundidade-1].listaProc[i];

                        var comp = await this.loadComplementares(p);
                        this.listaResultados = this.listaResultados.concat(comp);
                        this.stackProc[profundidade].listaProc = this.stackProc[profundidade].listaProc.concat(this.filtra(comp));

                        var sint = await this.loadSintetizados(p);
                        this.listaResultados = this.listaResultados.concat(sint);
                        this.stackProc[profundidade].listaProc = this.stackProc[profundidade].listaProc.concat(this.filtra(sint));

                        var sup = await this.loadSuplementares(p);
                        this.listaResultados = this.listaResultados.concat(sup); 
                        this.stackProc[profundidade].listaProc = this.stackProc[profundidade].listaProc.concat(this.filtra(sup));
                    }
                    profundidade++;
                    alert(JSON.stringify(this.stackProc))
                }
                
                this.listaResultados.sort(function (a, b) {
                        return a.codigo.localeCompare(b.codigo);
                });

                this.resultadosReady = true;
            }
            catch(erro){
                console.log(erro);
            }
        },

        filtra: function(lproc){
            return lproc.map(function(p){
                return p.codigo
            })
        },

        go: function(url){
            this.$router.push(url);
        }
    },

    created: function() {
        this.loadProcessos();
    },
}
</script>
