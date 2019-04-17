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
                                            <td> {{ props.item.nivel }} </td>
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
                { text: 'Relação', value: 'relacao' },
                { text: 'Distância', value: 'nivel'}
            ],
            niveis: 0,
            listaNiveis: [
                {text: "1 - Vizinhos imediatos", value: 1},
                {text: "2 - Vizinhos a 2 relações de distância", value: 2},
                {text: "3 - Vizinhos a 3 relações de distância", value: 3},
                {text: "4 - Vizinhos a 4 relações de distância", value: 4},
                {text: "5 - Vizinhos a 5 relações de distância", value: 5},
                {text: "6 - Vizinhos a 6 relações de distância", value: 6},
                {text: "7 - Vizinhos a 7 relações de distância", value: 7},
                {text: "8 - Vizinhos a 8 relações de distância", value: 8},
                {text: "Todos - irá tentar calcular o alcance total", value: 1000}
            ],
            stackProc: [],
            visitados: []
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

        loadComplementares: async function(p, profundidade){
            try{
                var response = await axios.get(lhost + "/api/classes/c" + p + "/procRel/eComplementarDe");
                return response.data.map(function (item) {
                    return {
                        codigo: item.codigo,
                        titulo: item.titulo,
                        relacao: "eComplementarDe",
                        nivel: profundidade
                    }
                    });
            }
           catch(erro){
                console.log(erro);
            }
        },

        loadSintetizados: async function(p, profundidade){
            try{
                var response = await axios.get(lhost + "/api/classes/c" + p + "/procRel/eSintetizadoPor");
                return response.data.map(function (item) {
                    return {
                        codigo: item.codigo,
                        titulo: item.titulo,
                        relacao: "eSintetizadoPor",
                        nivel: profundidade
                    }
                });
            }
           catch(erro){
                console.log(erro);
            }
        },

        loadSuplementares: async function(p, profundidade){
            try{
                var response = await axios.get(lhost + "/api/classes/c" + p + "/procRel/eSuplementoPara");
                return response.data.map(function (item) {
                    return {
                        codigo: item.codigo,
                        titulo: item.titulo,
                        relacao: "eSuplementoPara",
                        nivel: profundidade
                    }
                });
            }
           catch(erro){
                console.log(erro);
            }
        },

        calcRel: async function(){
            try{
                this.listaResultados = [];
                this.stackProc = [];
                this.visitados = [];
                this.stackProc.push({listaProc: [], nivel: 1});
                this.stackProc[0].listaProc.push(this.processo);
                this.visitados.push(this.processo);  // Processo inicial está no índice 0
                var profundidade = 1;
                var p;
                var stop = false;

                while(profundidade <= this.niveis){
                    this.stackProc.push({listaProc: [], nivel: profundidade+1});
                    for(var i=0; i < this.stackProc[profundidade-1].listaProc.length; i++){
                        p = this.stackProc[profundidade-1].listaProc[i];

                        var comp = await this.loadComplementares(p, profundidade);
                        if(comp.length > 0){
                            this.listaResultados = await this.juntaNovos(this.listaResultados, comp);
                            this.stackProc[profundidade].listaProc = this.stackProc[profundidade].listaProc.concat( await this.juntaNovosVisitas(this.visitados, this.filtra(comp), "comp"));
                        }
                        

                        var sint = await this.loadSintetizados(p, profundidade);
                        if(sint.length > 0){
                            this.listaResultados = await this.juntaNovos(this.listaResultados, sint);
                            this.stackProc[profundidade].listaProc = this.stackProc[profundidade].listaProc.concat( await this.juntaNovosVisitas(this.visitados, this.filtra(sint), "sint"));
                        }

                        var sup = await this.loadSuplementares(p, profundidade);
                        if(sup.length > 0){
                            this.listaResultados = await this.juntaNovos(this.listaResultados, sup);
                            this.stackProc[profundidade].listaProc = this.stackProc[profundidade].listaProc.concat( await this.juntaNovosVisitas(this.visitados, this.filtra(sup), "sup"));
                        }
                    }
                    profundidade++;
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

        juntaNovosVisitas: async function(visitados, candidatos, relacao){
            var res = [];
            for(var i=0; i < candidatos.length; i++){
                var index = visitados.indexOf(candidatos[i]);
                if(index == -1){
                    visitados.push(candidatos[i]);
                    res.push(candidatos[i]);
                }
                else if((index == 0) && (relacao != "comp")){
                    alert("Circularidade!!!");
                }
            }
            return res;
        },

        juntaNovos: async function(existentes, candidatos){
            for(var i=0; i < candidatos.length; i++){
                if(existentes.length > 0){
                    var index = existentes.findIndex(p => p.codigo == candidatos[i].codigo);
                    if(index == -1){
                        existentes.push(candidatos[i]);
                    }
                }
                else
                    existentes.push(candidatos[i]);
            }
            return existentes;
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
