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
                                <v-btn round color="info" @click="calcRel(processo); calculoTotal = false">Calcular</v-btn>
                            </v-flex>
                        </v-layout>

                        <v-layout>
                            <v-flex>
                                <v-btn round color="info" @click="calcRelTotal(); calculoTotal = true">Calcular todas as travessias de todos os processos</v-btn>
                            </v-flex>
                        </v-layout>
                        
                        <v-layout v-if="aCalcular">
                            <v-flex>
                                <v-btn round color="error" @click="cancelar()"
                                    >Cancelar Calculo</v-btn
                                >
                            </v-flex>
                        </v-layout>

                    </v-card-text>
                </v-card>
            </v-flex>
        </v-layout>

        <v-layout row wrap justify-center>
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
                                <p>A calcular a travessia...    Código: {{ processo }}</p>
                            </v-flex>
                        </v-layout>
                        <v-layout v-if="!fechoTotalCalculado">
                            <v-flex xs12>
                                {{ resultadosJSON }}
                            </v-flex>
                        </v-layout>
                        <v-layout wrap v-if="fechoTotalCalculado">
                            <v-flex xs12>
                                {{ resultadosJSON }}
                            </v-flex>
                        </v-layout>

                        <v-snackbar
                            v-model="fechoTotalCalculado"
                            :color="'success'"
                            :timeout="60000"
                        >
                            Cálculo total terminado:
                            Pode realizar outro cálculo.
                            <v-btn
                                dark flat
                                @click="fechoTotalCalculado = false"
                            >
                                Fechar
                            </v-btn>
                        </v-snackbar>

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

                        <v-snackbar
                            v-model="fechoCalculado"
                            :color="'success'"
                            :timeout="60000"
                        >
                            Cálculo terminado: {{ listaResultados.length }} processos.
                            Pode realizar outro cálculo.
                            <v-btn
                                dark flat
                                @click="fechoCalculado = false"
                            >
                                Fechar
                            </v-btn>
                        </v-snackbar>

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
            profundidade: 0,
            fechoCalculado: false,
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
                {text: "9 - Vizinhos a 9 relações de distância", value: 9},
                {text: "10 - Vizinhos a 10 relações de distância", value: 10},
                {text: "Todos - irá tentar calcular o alcance total", value: 1000}
            ],
            stackProc: [],
            visitados: [],
            calculoTotal: false,
            fechoTotalCalculado: false,
            resultadosJSON: [],
            aCalcular: false,
            // Todos os processos já visitados
            procVisitados: [],
            travessias: [],
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

        calcRel: async function(processo){
            try{
                console.log(processo)
                this.listaResultados = [];
                this.fechoCalculado = false;
                this.stackProc = [];
                this.visitados = [];
                this.stackProc.push({listaProc: [], nivel: 1});
                this.stackProc[0].listaProc.push(processo);
                this.visitados.push(processo);  // Processo inicial está no índice 0
                this.profundidade = 1;
                var p;
                var stop = false;

                while((this.profundidade <= this.niveis) && (this.stackProc[this.profundidade-1].listaProc.length > 0)){
                    this.stackProc.push({listaProc: [], nivel: this.profundidade+1});
                    for(var i=0; i < this.stackProc[this.profundidade-1].listaProc.length; i++){
                        p = this.stackProc[this.profundidade-1].listaProc[i];

                        console.log(p)

                        var comp = await this.loadComplementares(p, this.profundidade);
                        if(comp.length > 0){
                            this.listaResultados = await this.juntaNovos(this.listaResultados, comp);
                            this.stackProc[this.profundidade].listaProc = this.stackProc[this.profundidade].listaProc.concat( await this.juntaNovosVisitas(this.visitados, this.filtra(comp), "comp"));
                        }
                        

                        var sint = await this.loadSintetizados(p, this.profundidade);
                        if(sint.length > 0){
                            this.listaResultados = await this.juntaNovos(this.listaResultados, sint);
                            this.stackProc[this.profundidade].listaProc = this.stackProc[this.profundidade].listaProc.concat( await this.juntaNovosVisitas(this.visitados, this.filtra(sint), "sint"));
                        }

                        var sup = await this.loadSuplementares(p, this.profundidade);
                        if(sup.length > 0){
                            this.listaResultados = await this.juntaNovos(this.listaResultados, sup);
                            this.stackProc[this.profundidade].listaProc = this.stackProc[this.profundidade].listaProc.concat( await this.juntaNovosVisitas(this.visitados, this.filtra(sup), "sup"));
                        }
                    }
                    this.profundidade++;

                }
                
                this.listaResultados.sort(function (a, b) {
                        return a.codigo.localeCompare(b.codigo);
                });

                this.fechoCalculado = true;
                this.resultadosReady = true;
                return this.listaResultados
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
        },

        calcRelTotal: async function(){
            this.niveis = 1000;
            for( var i = 0; i < this.listaProcessos.length; i++ ){
                this.aCalcular = true
                this.processo = this.listaProcessos[i].value
                var resultado = await this.calcRel(this.listaProcessos[i].value)
                var lista = [];
                for( var j = 0; j < resultado.length; j ++){
                    lista.push(resultado[j].codigo)
                }
                this.travessias[this.processo] = lista;
                this.resultadosJSON.push({
                    processo: this.processo,
                    travessia: lista
                }) 
                console.log(this.travessias)
            }
            this.fechoTotalCalculado = true; 
            this.aCalcular = false;
        },
        cancelar: function(){
            window.location.reload()
        }
    },

    created: function() {
        this.loadProcessos();
    },
}
</script>
