<script setup>
import { ref, computed } from 'vue'

const produtos = ref([
  { id: 1, nome: 'Arroz 5kg', preco: 24.90, estoque: 12, quantidade: 1 },
  { id: 2, nome: 'Leite integral', preco: 5.49, estoque: 20, quantidade: 1 },
  { id: 3, nome: 'Óleo de soja', preco: 7.99, estoque: 8, quantidade: 1 },
  { id: 4, nome: 'Açúcar 1kg', preco: 4.50, estoque: 15, quantidade: 1 },
  { id: 5, nome: 'Feijão 1kg', preco: 8.90, estoque: 10, quantidade: 1 },
  { id: 6, nome: 'Macarrão 500g', preco: 4.99, estoque: 18, quantidade: 1 }
])

const pesquisa = ref('')
const reserva = ref([])
const mensagem = ref('')

const produtosFiltrados = computed(() =>
  produtos.value.filter(produto =>
    produto.nome.toLowerCase().includes(pesquisa.value.toLowerCase())
  )
)

const total = computed(() =>
  reserva.value.reduce((soma, produto) =>
    soma + produto.preco * produto.quantidade, 0
  )
)

function adicionar(produto) {
  if (!Number.isInteger(produto.quantidade) ||
      produto.quantidade < 1 ||
      produto.quantidade > produto.estoque) {
    mensagem.value = 'Informe uma quantidade válida e disponível.'
    return
  }

  const existente = reserva.value.find(item => item.id === produto.id)

  if (existente) {
    if (existente.quantidade + produto.quantidade > produto.estoque) {
      mensagem.value = 'Estoque insuficiente para essa quantidade.'
      return
    }
    existente.quantidade += produto.quantidade
  } else {
    reserva.value.push({
      id: produto.id,
      nome: produto.nome,
      preco: produto.preco,
      quantidade: produto.quantidade
    })
  }

  mensagem.value = `${produto.nome} adicionado à sua reserva.`
}

function remover(id) {
  reserva.value = reserva.value.filter(item => item.id !== id)
  mensagem.value = 'Produto removido da reserva.'
}

function confirmar() {
  if (reserva.value.length === 0) {
    mensagem.value = 'Adicione pelo menos um produto antes de confirmar.'
    return
  }

  mensagem.value = 'Reserva demonstrativa confirmada! Código: ANG-001.'
  reserva.value = []
}
</script>

<template>
  <div class="pagina">
    <header class="topo">
      <div class="marca">
        <span class="simbolo">A</span>
        <div>
          <h1>ANGELONI</h1>
          <p>SEMPRE PERTO DE VOCÊ</p>
        </div>
      </div>
      <span class="identificacao">CONSULTA E RESERVA DE PRODUTOS</span>
    </header>

    <main>
      <section class="boas-vindas">
        <p class="etiqueta">SUPERMERCADO ANGELONI</p>
        <h2>Suas compras com mais praticidade.</h2>
        <p>Consulte produtos, confira os preços e monte sua reserva.</p>
      </section>

      <section class="secao">
        <div class="titulo-secao">
          <div>
            <h2>Nossos produtos</h2>
            <p>Encontre o que você precisa.</p>
          </div>
          <span class="contador">{{ produtosFiltrados.length }} produtos</span>
        </div>

        <input
          v-model="pesquisa"
          class="pesquisa"
          type="text"
          placeholder="Buscar produto..."
          aria-label="Buscar produto"
        >

        <div v-if="produtosFiltrados.length" class="grade-produtos">
          <article v-for="produto in produtosFiltrados" :key="produto.id" class="card-produto">
            <div class="icone-produto">🛒</div>
            <h3>{{ produto.nome }}</h3>
            <p class="preco">R$ {{ produto.preco.toFixed(2).replace('.', ',') }}</p>
            <p class="estoque">{{ produto.estoque }} unidades disponíveis</p>

            <label :for="'quantidade-' + produto.id">Quantidade</label>
            <input
              :id="'quantidade-' + produto.id"
              v-model.number="produto.quantidade"
              class="quantidade"
              type="number"
              min="1"
              :max="produto.estoque"
            >
            <button class="botao" @click="adicionar(produto)">
              Adicionar à reserva
            </button>
          </article>
        </div>

        <p v-else class="aviso-vazio">Nenhum produto encontrado.</p>
      </section>

      <section class="secao reserva">
        <div class="titulo-secao">
          <div>
            <h2>Minha reserva</h2>
            <p>Confira os itens selecionados.</p>
          </div>
          <span class="contador">{{ reserva.length }} itens</span>
        </div>

        <p v-if="reserva.length === 0" class="aviso-vazio">
          Sua reserva está vazia. Adicione algum produto!
        </p>

        <div v-for="item in reserva" :key="item.id" class="item-reserva">
          <div>
            <strong>{{ item.nome }}</strong>
            <p>{{ item.quantidade }} unidade(s) × R$ {{ item.preco.toFixed(2).replace('.', ',') }}</p>
          </div>
          <div class="acoes-item">
            <strong>R$ {{ (item.preco * item.quantidade).toFixed(2).replace('.', ',') }}</strong>
            <button class="remover" @click="remover(item.id)">Remover</button>
          </div>
        </div>

        <div class="total">
          <span>Total estimado</span>
          <strong>R$ {{ total.toFixed(2).replace('.', ',') }}</strong>
        </div>

        <button class="botao confirmar" @click="confirmar">
          Confirmar reserva
        </button>

        <p v-if="mensagem" class="mensagem">{{ mensagem }}</p>
        <p class="observacao">
          Protótipo acadêmico. A reserva é demonstrativa e não é enviada ao supermercado.
        </p>
      </section>
    </main>

    <footer>
      <strong>SUPERMERCADO ANGELONI</strong>
      <p>Projeto acadêmico — Sistema de consulta e reserva de produtos</p>
    </footer>
  </div>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, Helvetica, sans-serif;
  background: #f5f6f3;
  color: #292d29;
}

button, input {
  font: inherit;
}

.topo {
  min-height: 86px;
  padding: 16px max(5%, calc((100% - 1100px) / 2));
  background: #176b3a;
  color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
}

.marca {
  display: flex;
  align-items: center;
  gap: 12px;
}

.simbolo {
  width: 43px;
  height: 43px;
  border: 2px solid white;
  border-radius: 50%;
  display: grid;
  place-items: center;
  font-size: 25px;
  font-weight: bold;
}

.marca h1 {
  margin: 0;
  font-size: 25px;
  letter-spacing: 2px;
}

.marca p {
  margin: 3px 0 0;
  font-size: 9px;
  letter-spacing: 2px;
}

.identificacao {
  font-size: 11px;
  letter-spacing: 1px;
  text-align: right;
}

main {
  width: min(1100px, 90%);
  margin: 0 auto;
}

.boas-vindas {
  padding: 48px 0 35px;
}

.etiqueta {
  color: #27834b;
  font-size: 12px;
  font-weight: bold;
  letter-spacing: 1.5px;
}

.boas-vindas h2 {
  margin: 10px 0;
  font-size: clamp(26px, 4vw, 36px);
  color: #205d38;
}

.boas-vindas > p:last-child,
.titulo-secao p {
  color: #737973;
}

.secao {
  margin-bottom: 35px;
}

.titulo-secao {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 15px;
  margin-bottom: 18px;
}

.titulo-secao h2 {
  color: #205d38;
  margin: 0 0 5px;
}

.titulo-secao p {
  margin: 0;
  font-size: 14px;
}

.contador {
  color: #176b3a;
  background: #e3f0e6;
  padding: 8px 12px;
  border-radius: 20px;
  font-size: 12px;
  white-space: nowrap;
}

.pesquisa {
  width: 100%;
  padding: 14px 16px;
  border: 1px solid #d8ded8;
  border-radius: 7px;
  margin-bottom: 20px;
  background: white;
  outline-color: #27834b;
}

.grade-produtos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 17px;
}

.card-produto {
  padding: 20px;
  background: white;
  border: 1px solid #e6e9e5;
  border-radius: 9px;
  box-shadow: 0 2px 7px #00000008;
}

.icone-produto {
  width: 43px;
  height: 43px;
  display: grid;
  place-items: center;
  background: #edf5ee;
  border-radius: 8px;
  font-size: 22px;
}

.card-produto h3 {
  margin: 15px 0 8px;
}

.preco {
  color: #176b3a;
  font-size: 21px;
  font-weight: bold;
  margin: 0 0 6px;
}

.estoque {
  color: #777;
  font-size: 12px;
  margin-bottom: 18px;
}

.card-produto label {
  display: block;
  font-size: 13px;
  margin-bottom: 6px;
}

.quantidade {
  width: 75px;
  padding: 8px;
  border: 1px solid #d8ded8;
  border-radius: 5px;
}

.botao {
  width: 100%;
  border: 0;
  border-radius: 6px;
  padding: 12px;
  background: #176b3a;
  color: white;
  font-weight: bold;
  cursor: pointer;
  margin-top: 13px;
}

.botao:hover {
  background: #10532c;
}

.reserva {
  padding: 23px;
  background: white;
  border: 1px solid #e6e9e5;
  border-radius: 9px;
}

.item-reserva {
  padding: 14px 0;
  border-bottom: 1px solid #edf0ec;
  display: flex;
  justify-content: space-between;
  gap: 15px;
}

.item-reserva p {
  margin: 5px 0 0;
  color: #777;
  font-size: 13px;
}

.acoes-item {
  text-align: right;
}

.remover {
  display: block;
  border: 0;
  padding: 5px 0;
  color: #b13d35;
  background: transparent;
  cursor: pointer;
  font-size: 12px;
}

.total {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 20px;
  font-size: 17px;
}

.total strong {
  color: #176b3a;
  font-size: 22px;
}

.confirmar {
  max-width: 280px;
}

.aviso-vazio {
  color: #777;
  padding: 15px 0;
}

.mensagem {
  color: #176b3a;
  font-weight: bold;
  margin-bottom: 0;
}

.observacao {
  color: #888;
  font-size: 11px;
  margin-top: 15px;
}

footer {
  padding: 25px 15px;
  background: #174f30;
  color: white;
  text-align: center;
  font-size: 13px;
}

footer p {
  margin-bottom: 0;
  opacity: 0.8;
}

@media (max-width: 600px) {
  .topo {
    align-items: flex-start;
    flex-direction: column;
  }

  .identificacao {
    text-align: left;
  }

  .boas-vindas {
    padding-top: 32px;
  }

  .reserva {
    padding: 16px;
  }
}
</style>
