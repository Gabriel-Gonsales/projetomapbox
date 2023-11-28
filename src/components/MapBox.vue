<template>
    <div ref="mapContainer" class="map-container"></div>
  </template>
  
  <script>
  import mapboxgl from "mapbox-gl";
  mapboxgl.accessToken = 'pk.eyJ1IjoiZ2FicmllbGdvbnNhbGVzIiwiYSI6ImNsb2hpdnR3YjAwMXEyaW81MnRheHZrb3cifQ.mWhh_5I0ESQAuPFGj-xCAQ';
  
  export default {
    props: ["modelValue"],
  
    data: () => (
        {
          map: null,
          mediasPorArea: {},
          areasMap :{},
          filtroSelecionado: -1,
          valoresPond: []
        }
      ),
  
    mounted() {
      this.getDadosSetores();
      this.calcularMedia()
      this.valoresPond = [];
    },
  
    unmounted() {
      this.map.remove();
      this.map = null;
    },
  
    watch: {
      modelValue(next) {
        const curr = this.getLocation();
        const map = this.map;
  
        if (curr.lng != next.lng || curr.lat != next.lat)
          map.setCenter({ lng: next.lng, lat: next.lat });
        if (curr.pitch != next.pitch) map.setPitch(next.pitch);
        if (curr.bearing != next.bearing) map.setBearing(next.bearing);
        if (curr.zoom != next.zoom) map.setZoom(next.zoom);
      },
    },
  
    methods: {
      buildMap(){
        const { lng, lat, zoom, bearing, pitch } = this.modelValue;
  
        const map = new mapboxgl.Map({
          container: this.$refs.mapContainer,
          style: "mapbox://styles/mapbox/streets-v12",
          center: [lng, lat],
          bearing,
          pitch,
          zoom,
        });

        const updateLocation = () =>
          this.$emit("update:modelValue", this.getLocation());

        map.on("move", updateLocation);
        map.on("zoom", updateLocation);
        map.on("rotate", updateLocation);
        map.on("pitch", updateLocation);
        map.on('load', () => {
              //Aqui eu coloco a fonte de dados que é um geojson
              map.addSource('zonasBauru', {
                  'type': 'geojson',
                  'data': require('@/assets/setoresBauru.geojson')
              });
              map.addLayer({
              'id': 'fill-layer',
              'type': 'fill',
              'source': 'zonasBauru',
              'paint': {
                'fill-color': [
                  'match',
                  ['get', 'name'],
                  // Generate the case statements dynamically for each area code
                  ...Array.from({ length: 763 }, (_, i) => [
                    `350600305000${String(i + 1).padStart(3, '0')}`, this.calcularCor(`350600305000${String(i + 1).padStart(3, '0')}`)
                  ]).flat(),
                  '#c9fff9' // Default color if there is no match
                ],
                'fill-opacity': 0.4
              }
            });
              //Esse trecho aqui coloca a linha preta de contorno de cada setor
              map.addLayer({
                'id': 'outline',
                'type': 'line',
                'source': 'zonasBauru',
                'layout': {},
                'paint': {
                  'line-color': '#030',
                  'line-width': 0.5
                }
              });
                  //Quando clico em cada zona mostra a tabela que tem nela (talvez eu tire isso)
              map.on('click', 'zonaCensitaria', (e) => {
                const description = e.features[0].properties.description;

                new mapboxgl.Popup()
                  .setLngLat(e.lngLat)
                  .setHTML(description)
                  .addTo(map);
              });

              //só muda o cursor pra pointer quando estou sob a figura
              map.on('mouseenter', 'zonaCensitaria', () => {
                map.getCanvas().style.cursor = 'pointer';
              });

              // muda o cursor de volta
              map.on('mouseleave', 'zonaCensitaria', () => {
                map.getCanvas().style.cursor = '';
              });
          });
        
        this.map = map;
      },
      getLocation() {
        return {
          ...this.map.getCenter(),
          bearing: this.map.getBearing(),
          pitch: this.map.getPitch(),
          zoom: this.map.getZoom(),
        };
      },
      getDadosSetores(){
        var requireSetores = require('@/assets/Dados_Setores_Bauru.json');

          // Iterar sobre cada objeto no JSON e organizá-los por área de ponderação
          requireSetores.forEach(obj => {
            const area = obj["ÁREA DE PONDERAÇÃO"];

            // Verificar se já existe um array para a área ou criar um novo
            if (!this.areasMap[area]) {
              this.areasMap[area] = [];
            }

            // Adicionar o objeto ao array correspondente
            this.areasMap[area].push(obj);
          });

          // Agora, você terá um objeto onde as chaves são as áreas de ponderação
          // e os valores são arrays de objetos correspondentes a essa área
      },
      calcularCor(areaPonderacao){
        let media = this.mediasPorArea[areaPonderacao]
        if(this.valoresPond.length == 0) return '#c9fff9'
        if(media){
          if(media <= this.valoresPond[0]) return '#e6003d'
          if(media <= this.valoresPond[1]) return '#f58002'
          if(media <= this.valoresPond[2]) return '#f5b402'
          if(media <= this.valoresPond[3]) return '#c4f502'
          if(media <= this.valoresPond[4]) return '#80f502'
          if(media <= this.valoresPond[5]) return '#09ba2a'
          else return '#e100ff'
        }
        return '#c9fff9'
      },
      calcularMedia(filtro, valores){
        if(valores){
          this.valoresPond = valores;
        }
        this.filtroSelecionado = filtro;
        this.map = null;
        // Vamos assumir que 'areasMap' já foi criado conforme o exemplo anterior

        // Função para calcular a média de um campo específico em um array de objetos
        const calcularMedia = (arr, campo) => {
          const valoresValidos = arr
            .map(obj => obj[campo])
            .filter(valor => valor !== null && valor !== undefined && valor !== "");

          if (valoresValidos.length === 0) {
            return 0; // Ou qualquer valor padrão desejado se não houver valores válidos
          }

          const total = valoresValidos.reduce((soma, valor) => soma + parseInt(valor, 10), 0);
          return total / valoresValidos.length;
        };
        
        // Objeto para armazenar médias por área de ponderação
        this.mediasPorArea;
        if(filtro && valores.length != 0){
          switch(filtro){
            case '0':
              for (const area in this.areasMap) {
                const objetosNaArea = this.areasMap[area];
                const media = calcularMedia(objetosNaArea, "RENDIMENTO MENSAL DOMICILIAR EM JULHO DE 2010");
                this.mediasPorArea[area] = media;
              }
            break;
            case '1':
              for (const area in this.areasMap) {
                const objetosNaArea = this.areasMap[area];
                const media = calcularMedia(objetosNaArea, "RENDIMENTO DOMICILIAR PER CAPITA EM JULHO DE 2010");
                this.mediasPorArea[area] = media;
              }
            break;
            case '2':
              for (const area in this.areasMap) {
                const objetosNaArea = this.areasMap[area];
                let contagemPerCapitaMaiorQueDois = 0;

                // Iterar sobre os objetos na área
                for (const objeto of objetosNaArea) {
                    const rendimentoPerCapita = parseInt(objeto["RENDIMENTO DOMICILIAR PER CAPITA, EM Nº DE SALÁRIOS MÍNIMOS, EM JULHO DE 2010"], 10);
                    // Verificar se o rendimento per capita é maior que "00200000"
                    if (rendimentoPerCapita > 20) {
                        contagemPerCapitaMaiorQueDois++;
                    }
                }

                // Armazenar a contagem na estrutura de dados apropriada (se necessário)
                this.mediasPorArea[area] = contagemPerCapitaMaiorQueDois;
              }
            break;
          }
        }
        else{
          // Iterar sobre o objeto 'areasMap' e calcular a média para cada área de ponderação
          for (const area in this.areasMap) {
            const objetosNaArea = this.areasMap[area];
            const media = calcularMedia(objetosNaArea, "RENDIMENTO DOMICILIAR PER CAPITA EM JULHO DE 2010");
            this.mediasPorArea[area] = media;
          }
        }
        this.buildMap();
        // Agora, 'mediasPorArea' conterá as médias para cada área de ponderação, tratando valores nulos ou vazios
      }
    },
  };
  </script>
  
  <style>
  .map-container {
    flex: 1;
  }
  </style>