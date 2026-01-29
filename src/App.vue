<script setup>
import { ref, reactive, onMounted } from 'vue'
import './style.css'

const icones = [
  { nome: 'Maranhão', img: new URL('./assets/Maranhão.png', import.meta.url).href },
  { nome: 'Piauí', img: new URL('./assets/Piaui.png', import.meta.url).href },
  { nome: 'Ceará', img: new URL('./assets/Ceara.png', import.meta.url).href },
  { nome: 'Rio Grande do Norte', img: new URL('./assets/rio_grande.png', import.meta.url).href },
  { nome: 'Paraíba', img: new URL('./assets/Paraiba.png', import.meta.url).href },
  { nome: 'Pernambuco', img: new URL('./assets/Pernambuco.png', import.meta.url).href },
  { nome: 'Alagoas', img: new URL('./assets/Alagoas.png', import.meta.url).href },
  { nome: 'Sergipe', img: new URL('./assets/sergipe.png', import.meta.url).href },
  { nome: 'Bahia', img: new URL('./assets/Bahia.png', import.meta.url).href },
]

const iconeMutante = icones[Math.floor(Math.random() * icones.length)]
const jogoGanho = ref(false)
const escolhas = ref([])

const cartas = reactive(
  [...icones, ...icones]
    .map((estado, i) => ({
      id: i,
      valor: estado,
      estaVirada: false,
      foiEncontrada: false,
      tipo: estado.nome === iconeMutante.nome ? 'mutante' : 'normal'
    }))
    .sort(() => Math.random() - 0.5)
)

const virarCarta = (index) => {
  const carta = cartas[index]
  if (carta.estaVirada || carta.foiEncontrada || escolhas.value.length === 2) return

  carta.estaVirada = true
  escolhas.value.push(index)

  if (carta.tipo === 'mutante' && escolhas.value.length === 1) {
    setTimeout(() => {
      aplicarEfeitoMutante()
      carta.estaVirada = false
      escolhas.value = []
    }, 700)
    return
  }

  if (escolhas.value.length === 2) {
    checarPar()
  }
}

const checarPar = () => {
  const [i1, i2] = escolhas.value

  if (cartas[i1].valor.nome === cartas[i2].valor.nome) {
    cartas[i1].foiEncontrada = true
    cartas[i2].foiEncontrada = true
    escolhas.value = []
    verificarVitoria()
  } else {
    setTimeout(() => {
      cartas[i1].estaVirada = false
      cartas[i2].estaVirada = false
      escolhas.value = []
    }, 1000)
  }
}

const aplicarEfeitoMutante = () => {
  const indicesDisponiveis = cartas
    .map((c, i) => (!c.foiEncontrada && !c.estaVirada ? i : null))
    .filter(i => i !== null)

  const valores = indicesDisponiveis.map(i => cartas[i].valor)
  valores.sort(() => Math.random() - 0.5)

  indicesDisponiveis.forEach((posicao, k) => {
    cartas[posicao].valor = valores[k]
    cartas[posicao].tipo = valores[k].nome === iconeMutante.nome ? 'mutante' : 'normal'
  })

  alert(`A Máscara de ${iconeMutante.nome} bagunçou as identidades!`)
}

const verificarVitoria = () => {
  if (cartas.every(carta => carta.foiEncontrada)) {
    jogoGanho.value = true
  }
}

const reiniciarJogo = () => {
  window.location.reload()
}
</script>

<template>
  <main>
    <h1>Máscaras do Nordeste 🎭</h1>
    <p>Cuidado com a máscara mutante: <strong>{{ iconeMutante.nome }}</strong></p>

    <div v-if="jogoGanho" class="vitoria-overlay">
      <div class="vitoria-card">
        <h2>Parabéns!</h2>
        <p>Você dominou as identidades do Nordeste!</p>
        <button @click="reiniciarJogo">Jogar Novamente</button>
      </div>
    </div>

    <div class="tabuleiro">
      <div 
        v-for="(carta, index) in cartas" 
        :key="carta.id"
        class="carta"
        :class="{ 
          'aberta': carta.estaVirada || carta.foiEncontrada,
          'mutante-style': carta.tipo === 'mutante' && (carta.estaVirada || carta.foiEncontrada)
        }"
        @click="virarCarta(index)"
      >
        <template v-if="carta.estaVirada || carta.foiEncontrada">
          <img :src="carta.valor.img" :alt="carta.valor.nome" class="img-mask" />
          <span>{{ carta.valor.nome }}</span>
        </template>
        
        <template v-else>
          <div class="verso">?</div>
        </template>
      </div>
    </div>
  </main>
</template>

