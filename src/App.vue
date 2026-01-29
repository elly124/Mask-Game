<script setup>
// Importa funções reativas e ciclo de vida do Vue
import { ref, reactive, onMounted } from 'vue'

// Importa o CSS do componente
import './style.css'

/**
 * Lista de ícones (máscaras) representando estados do Nordeste
 * Cada item possui um nome e o caminho da imagem
 */
const icones = [
  { nome: 'Maranhão', img: new URL('/Maranhao.png', import.meta.url).href },
  { nome: 'Piauí', img: new URL('/Piaui.png', import.meta.url).href },
  { nome: 'Ceará', img: new URL('/Ceara.png', import.meta.url).href },
  { nome: 'Rio Grande do Norte', img: new URL('/Rio_Grande.png', import.meta.url).href },
  { nome: 'Paraíba', img: new URL('/Paraiba.png', import.meta.url).href },
  { nome: 'Pernambuco', img: new URL('/Pernambuco.png', import.meta.url).href },
  { nome: 'Alagoas', img: new URL('/Alagoas.png', import.meta.url).href },
  { nome: 'Sergipe', img: new URL('/sergipe.png', import.meta.url).href },
  { nome: 'Bahia', img: new URL('/Bahia.png', import.meta.url).href },
]

/**
 * Escolhe aleatoriamente um ícone para ser a "máscara mutante"
 */
const iconeMutante = icones[Math.floor(Math.random() * icones.length)]

// Indica se o jogo foi ganho
const jogoGanho = ref(false)

// Armazena os índices das cartas escolhidas na rodada atual
const escolhas = ref([])

/**
 * Criação das cartas do jogo:
 * - Duplica os ícones
 * - Define propriedades de controle
 * - Embaralha as cartas
 */
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

/**
 * Função chamada ao clicar em uma carta
 */
const virarCarta = (index) => {
  const carta = cartas[index]

  // Impede virar cartas inválidas
  if (carta.estaVirada || carta.foiEncontrada || escolhas.value.length === 2) return

  // Vira a carta e registra a escolha
  carta.estaVirada = true
  escolhas.value.push(index)

  // Se for carta mutante e for a primeira escolha
  if (carta.tipo === 'mutante' && escolhas.value.length === 1) {
    setTimeout(() => {
      aplicarEfeitoMutante()
      carta.estaVirada = false
      escolhas.value = []
    }, 700)
    return
  }

  // Se duas cartas foram escolhidas, verifica o par
  if (escolhas.value.length === 2) {
    checarPar()
  }
}

/**
 * Verifica se as duas cartas escolhidas formam um par
 */
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

/**
 * Aplica o efeito da máscara mutante:
 * embaralha as identidades das cartas não encontradas
 */
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

/**
 * Verifica se todas as cartas foram encontradas
 */
const verificarVitoria = () => {
  if (cartas.every(carta => carta.foiEncontrada)) {
    jogoGanho.value = true
  }
}

/**
 * Reinicia o jogo recarregando a página
 */
const reiniciarJogo = () => {
  window.location.reload()
}
</script>

<template>
  <main>
    <!-- Título do jogo -->
    <h1>Máscaras do Nordeste 🎭</h1>

    <!-- Informação da máscara mutante -->
    <p>Cuidado com a máscara mutante: <strong>{{ iconeMutante.nome }}</strong></p>

    <!-- Tela de vitória -->
    <div v-if="jogoGanho" class="vitoria-overlay">
      <div class="vitoria-card">
        <h2>Parabéns!</h2>
        <p>Você dominou as identidades do Nordeste!</p>
        <button @click="reiniciarJogo">Jogar Novamente</button>
      </div>
    </div>

    <!-- Tabuleiro do jogo -->
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
        <!-- Frente da carta -->
        <template v-if="carta.estaVirada || carta.foiEncontrada">
          <img :src="carta.valor.img" :alt="carta.valor.nome" class="img-mask" />
          <span>{{ carta.valor.nome }}</span>
        </template>
        
        <!-- Verso da carta -->
        <template v-else>
          <div class="verso">?</div>
        </template>
      </div>
    </div>
  </main>
</template>
