<script>
  import Map from "./components/MapBox.vue";
  import "../node_modules/mapbox-gl/dist/mapbox-gl.css"

  export default {
    components: {
      Map,
    },

    data: () => ({
      location: {
        lng: -49.071457542402435,
        lat: -22.316021572407285,
        bearing: 0,
        pitch: 0,
        zoom: 11.9
      },
      filtroSelecionado: 'Selecione um filtro',
      mensagem: '',
      titulo: '',
      valoresPond: []
    }),
    methods:{
      calculaComFiltro(escolhendo = true){
        if(this.filtroSelecionado == 0 && escolhendo){
          this.mensagem = "A média da renda domiciliar mensal permite compreender as condições socioeconômicas da população, isso inclui a capacidade de atrair clientes com maior poder de compra, segmentação eficaz do mercado, estabilidade financeira, margens de lucro mais elevadas, menor concorrência, investimentos em marketing mais eficazes, oportunidades de expansão futura e a possibilidade de construir a fidelidade do cliente.";
          this.valoresPond = [70000,140000,440000,660000,760000,890000];
          this.titulo = 'Média da renda mensal (x100)'
        } 
        else if(this.filtroSelecionado == 1  && escolhendo){
          this.mensagem = "A média da renda domiciliar mensal per capita permite compreender as condições socioeconômicas da população e como ela está distribuída nas famílias, mas não reflete como a renda está distribuída na área censitária como um todo.";
          this.valoresPond = [70000,644000,1000000,1200000,1600000,12000000];
          this.titulo = 'Média da renda mensal per capita(x100)'
        }
        else if(this.filtroSelecionado == 2  && escolhendo){ this.mensagem = "Renda familiar per capita superior a 2 salários mínimos também é um importante indicador de poder de compra na região.";
        this.valoresPond = [0,10,20,30,40,50]
        this.titulo = 'Quantidade média de domicílios onde a renda é maior que 2 salários mínimos'
        }
        else if(escolhendo){
          this.mensagem = '';
          this.valoresPond = [];
        }
        this.$refs.map.calcularMedia(this.filtroSelecionado,this.valoresPond);
      }
    }
  };
</script>

<template>
  <div id="layout">
    <div id="sidebar">
      <button @click="location = { lng: -49.071457542402435, lat: -22.316021572407285, zoom: 11.9, pitch: 0, bearing: 0 }">Centralizar mapa</button>
    </div>
    <div id="legenda" v-if="valoresPond.length > 0">
      <h4>{{ titulo }}</h4>
      Inviável
      <div class="caixaColor" style="background-color: #e6003d;"><input @keyup.enter="calculaComFiltro(false)" v-model="valoresPond[0]" class="inputColors"></div>
      Ruim
      <div class="caixaColor" style="background-color: #f58002;"><input @keyup.enter="calculaComFiltro(false)"  v-model="valoresPond[1]" class="inputColors"></div>
      Requer atenção
      <div class="caixaColor" style="background-color: #f5b402;"><input @keyup.enter="calculaComFiltro(false)"  v-model="valoresPond[2]" class="inputColors"></div>
      Médio
      <div class="caixaColor" style="background-color: #c4f502;" ><input @keyup.enter="calculaComFiltro(false)"  v-model="valoresPond[3]" class="inputColors"></div>
      Bom
      <div class="caixaColor" style="background-color: #80f502;"><input @keyup.enter="calculaComFiltro(false)"  v-model="valoresPond[4]" class="inputColors"></div>
      Muito bom
      <div class="caixaColor" style="background-color: #09ba2a;"><input @keyup.enter="calculaComFiltro(false)"  v-model="valoresPond[5]" class="inputColors"></div>
      Extraordinário
      <div class="caixaColor" style="background-color: #e100ff;"><span>Maior que anteriores</span></div>
    </div>
    <div id="descricao">
      {{ mensagem }}
    </div>
    <div id="ponderacao">
      <select v-model="this.filtroSelecionado" @change="calculaComFiltro">
        <option selected>Selecione um filtro</option>
        <option value="0">Rendimento mensal domiciliar em julho de 2010</option>
        <option value="1">Rendimento domiciliar per capita em julho de 2010</option>
        <option value="2">Rendimento domiciliar per capita acima de 2 salários mínimos</option>
      </select>
    </div>
    <Map ref="map" v-model="location"/>
  </div>
</template>

<style>
  body{
    margin: 0;
    padding: 0;
  }
  select{
    max-width: 10vw;
  }
  #layout {
    flex: 1;
    display: flex;
    height: 100vh;
    width: 100vw;
    padding: 0;
  }
  .caixaColor{
    border: 1px solid black;
    min-height: 12px;
    min-width: 12px;
  }
  .inputColors{
    max-width: 50px;
    max-height: 10px;
  }
  #sidebar {
    background-color: rgba(35, 55, 75, 0.9);
    color: #fff;
    padding: 6px 12px;
    font-family: monospace;
    z-index: 1;
    position: absolute;
    top: 0;
    left: 0;
    margin: 12px;
    border-radius: 4px;
  }
  #legenda{
    background-color: rgba(35, 55, 75, 0.9);
    color: #fff;
    padding: 6px 12px;
    font-family: monospace;
    z-index: 1;
    margin: 12px;
    max-width: 10vw;
    margin-bottom: 30px;
    align-self: flex-end;
    border-radius: 4px;
    position: absolute;
    bottom: 0;
    right: 0;
  }
  #descricao{
    background-color: rgba(35, 55, 75, 0.9);
    color: #fff;
    padding: 6px 12px;
    font-family: monospace;
    z-index: 1;
    margin: 12px;
    align-self: flex-end;
    border-radius: 4px;
    position: absolute;
    top: 0;
    right: 0;
    max-width: 50vw;
  }
  #ponderacao{
    background-color: rgba(35, 55, 75, 0.9);
    color: #fff;
    padding: 6px 12px;
    font-family: monospace;
    z-index: 1;
    margin: 12px;
    margin-bottom: 10vh;
    align-self: flex-end;
    border-radius: 4px;
    position: absolute;
    bottom: 0;
    left: 0;
  }
</style>