<template>
  <q-page>

    <q-toolbar class="text-primary bingo-title">
      <q-toolbar-title style="font-size: 3rem;">
        Bingo Biblico IBVN
      </q-toolbar-title>

      <!-- <q-img style="width: 350px;" src="~assets/title-bingo.png" /> -->

        <!-- <vue-particles
          id="tsparticles"
          :particlesInit="particlesInit"
          :particlesLoaded="particlesLoaded"
          url="http://foo.bar/particles.json"
        /> -->
    </q-toolbar>

    <h5></h5>

    <div>
        <q-btn v-if="canSort" size="lg" @click="randomico"  :color="'blue'" round :label="numero" />
    </div>

    <div class="cartelas q-mb-lg">
      <div style="margin: 10px;" :key="item.id" v-for="(item, i) in bingo">
        <q-btn @click="handleSelected(item, i)" :color="item.selected? 'blue': 'white text-grey'" round :label="item.id" />
      </div>
    </div>

    <q-dialog v-model="showCartelas">
      <q-card style="border-radius: 20px">

          <q-toolbar class="text-primary">
            <q-toolbar-title>
              Cartelas
            </q-toolbar-title>

            <q-btn size="sm" class="float-right" icon="mdi-air-filter" rounded label="Gerar PDF" no-caps color="primary" :loading="loading"  @click="generatePDF" />
            <q-linear-progress style="width: 50%; margin: 0 auto" v-if="loading" stripe rounded size="12px" :value="progress" color="warning" class="q-mt-sm text-center" />

          </q-toolbar>

        <q-card-section class="cartelas q-pt-none" style="height: 400px; overflow: auto">
          <div class="cartela" :ref="`cartela-${numCartela}`" :key="'ca-' + numCartela" v-for="(cartela, numCartela) in cartelas">
            <span class="item header">Bingo Bíblico - Confra Jovens IBVN</span>

            <div class="numero" :key="'be-' + item" v-for="item in cartela">
              <span v-if="isDebug">{{bingo[item].id}}</span>
              <q-img style="width: 50px;" v-if="bingo[item]?.image" :src="`/bingo/${bingo[item].image}`" />
            </div>
            <span class="item footer"><span style="float: right; font-size: .7rem">{{numCartela}}</span></span>
          </div>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn rounded flat label="Fechar" no-caps color="primary"  v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="showQuestions" persistent>
      <q-card style="border-radius: 20px;">
        <q-card-section class="cartelas q-pt-none">
            <div v-if="selectedItem.id" class="text-center q-pa-md " style="border-radius: 1rem;">
              <span class="block" style="font-size: 2rem">{{selectedItem.question}}</span>

              <div class="text-center q-mt-md">
                <div class="full-width flex flex-center">
                  <transition-group
                    appear
                    enter-active-class="animated scale"
                    leave-active-class="animated scale"
                  >
                    <q-img class="block q-mb-md" v-if="selectedItem?.image && selectedItem.show" style="width: 135px;" :src="`/bingo/${selectedItem.image}`" />
                  </transition-group>
                </div>

                <transition-group
                  enter-active-class="animated fadeIn"
                  leave-active-class="animated fadeOut"
                >
                  <span v-if="selectedItem?.image && selectedItem.show" class="blog" style="font-size: 1.1rem">{{selectedItem.answer}}</span>
                </transition-group>
              </div>
            </div>
        </q-card-section>

        <q-card-actions align="right" class="q-mb-xs">
          <q-btn v-if="!selectedItem.show" icon="mdi-bell-ring-outline" @click="selectedItem.show = true" rounded color="green" no-caps label="Revelar!" />
          <q-btn class="float-right q-mr-sm" v-if="selectedItem.show" @click="selectedItem.show = false" rounded outline  size="sm" color="red" no-caps label="Esconder" />
          <q-btn v-if="selectedItem.show" icon="mdi-party-popper"  rounded  label="Avançar" no-caps color="green" v-close-popup  />
          <q-btn v-else @click="cleanSelection(selectedItem, selectedItem.id -1)" flat rounded dense label="fechar" no-caps color="grey" v-close-popup  />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <div class="flex full-width q-mb-lg">
      <div style="margin: 10px;" :key="item.id" v-for="(item) in bingo.filter(el => el.selected && el.show)">
        <q-img style="width: 45px;" v-if="item?.image" :src="`/bingo/${item.image}`" />
      </div>
    </div>

    <div style="bottom: 0" class="fixed q-ma-sm bg-white">
      <q-toggle size="xs" dense class="q-mr-md" v-model="canSort" label="com sorteio" />
      <q-btn rounded color="green" :flat="!checaQuemVenceu.length" no-caps size="xs" dense class="q-mr-md" @click="showGanhador = !showGanhador" label="Revelar vencedores" />

      <div v-if="showGanhador" class="float-right">
        Cartelas Vencedoras:
        <q-chip :key="cartela" v-for="cartela in checaQuemVenceu.map( el => 'nº ' + el)">{{cartela}}</q-chip>
      </div>
    </div>

  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import { bingoEnum } from 'src/enum/bingo.js'
import { cards } from 'src/enum/cards.js'

import { jsPDF as JsPDF } from 'jspdf'
import html2canvas from 'html2canvas'

export default defineComponent({
  name: 'IndexPage',

  data () {
    return {
      bingo: bingoEnum,
      cards,
      showGanhador: false,
      isDebug: false,
      numero: 0,
      showCartelas: false,
      canSort: false,
      showQuestions: false,
      selectedItem: {},
      loading: false,
      getGanhador: {},
      progress: 0
    }
  },

  watch: {
    bingo: {
      handler (value) {
        if (value.filter(el => el.selected).length === 0) {
          this.selectedItem = {}
        }
      },
      deep: true
    }
  },

  computed: {
    cartelas () {
      return this.cards.map(s => s.split('-').map(Number).sort((a, b) => a - b))
    },

    itensSelecionado () {
      return this.bingo.filter(el => el.selected && el.show)
    },

    checaQuemVenceu () {
      if (this.itensSelecionado.length < 12) {
        return []
      }

      const a = this.itensSelecionado.map(el => el.id)
      const b = this.cards.map(el => el.split('-').map(el => +el))

      const isSubarray = (superArray, subArray) => {
        return subArray.every(val => superArray.includes(val))
      }

      const ganhadores = []

      for (let i = 0; i < b.length; i++) {
        if (isSubarray(a, b[i])) {
          ganhadores.push(i)
        }
      }

      return ganhadores
    }
  },

  methods: {
    randomico () {
      this.numero = Math.floor(Math.random() * 40)
    },

    async generatePDF () {
      this.loading = true
      const pdf = new JsPDF()
      let posY = 10

      this.progress = 0

      for (let i = 0; i < this.cartelas.length; i++) {
        const cartelaElement = this.$refs[`cartela-${i}`][0] // Acessando o elemento através da ref
        const canvas = await html2canvas(cartelaElement, { scale: 2 })

        const imgData = canvas.toDataURL('image/png')

        // const pageWidth = 210 // Largura da página A4 em mm
        const imgWidth = 190 // Deixamos uma margem de 10mm em cada lado
        const imgHeight = (canvas.height * imgWidth) / canvas.width

        pdf.addImage(imgData, 'PNG', 10, posY, imgWidth, imgHeight)

        // posY += canvas.height * (210 / canvas.width) // Ajuste da altura, considerando proporção e largura fixa de 210 para o tamanho A4.
        posY += imgHeight + 10 // Adiciona um espaço de 10mm entre as cartelas

        if (posY + imgHeight > 287 && i < this.cartelas.length - 1) { // 287mm deixa uma margem de 10mm na parte inferior da página.
          pdf.addPage()
          posY = 10 // Reseta a posição Y para o topo da nova página
        }

        this.progress = (i + 1) / this.cartelas.length * 100 / 100
      }

      pdf.save('cartela-bingo.pdf')

      this.loading = false
      this.showCartelas = false
    },

    handleSelected (item, i) {
      if (this.bingo[i].selected) {
        this.bingo[i].show = false
        this.bingo[i].selected = false
        this.selectedItem = item
      } else {
        if (this.bingo.filter(el => el.show).length === this.bingo.filter(el => el.selected).length) {
          this.bingo[i].selected = true
          this.selectedItem = item
          this.showQuestions = true
        }
      }
    },

    cleanSelection (item, i) {
      this.handleSelected(item, i)
    }
  }
})
</script>

<style lang="scss">
@font-face {
  font-family: "Agente Orange";
  src: url("src/css/fonts/agent-orange.TTF") format("truetype");
}

@font-face {
  font-family: "Cassino Shadow";
  src: url("src/css/fonts/CasinoShadow.ttf") format("truetype");
}

.bingo-title {
  font-family: "Cassino Shadow", sans-serif;
}

.teste {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: grey;
  margin-top: 60px;
}

body {
  background-color: white;
  font-family: Arial, sans-serif;
  text-align: center;
  padding: 20px;
}

.header{
  grid-area: header;
  font-size: 1.3rem
}

.footer{
  grid-area: footer;
}

.cartelas {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.cartela {
  border-radius: 10px;
  margin: 10px;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  width: 800px;
  gap: 15px;

  padding: 10px;
  padding-top: 15px;
  padding-left: 15px;
  padding-right: 15px;
  border: 1px solid grey;
  grid-template-areas: "header header header header"
                       "item item item item"
                       "item item item item"
                       "item item item item"
                       "footer footer footer footer";
  // box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
}

.numero {
  border: 1px solid grey;
  border-radius: 12px;
  padding: 10px;
  text-align: center;
}
</style>
