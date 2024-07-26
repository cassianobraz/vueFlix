<script setup>
import { computed, ref, onMounted, watch } from "vue";

const filmes = ref([])

const adicionandoFilme = ref(false)

const nomeFilme = ref("")
const imagemFilme = ref("")
const lancamentoFilme = ref("")
const generoFilme = ref("")
const salvarFilme = () => {
  filmes.value.push({
    nome: nomeFilme.value,
    lancamento: lancamentoFilme.value,
    genero: generoFilme.value,
    imagem: imagemFilme.value
  })

  fecharCriacaodeFilme()
}

const fecharCriacaodeFilme = () => {
  nomeFilme.value = ""
  lancamentoFilme.value = ""
  generoFilme.value = ""
  imagemFilme.value = ""

  adicionandoFilme.value = false
}

const definirLike = (index, like) => {
  filmes.value[ index ].like = like
}

const excluirFilme = (index) => {
  const texto = `Tem certeza que deseja excluir o filme ${filmes.value[ index ].nome}?`
  if (confirm(texto)) {
    filmes.value.splice(index, 1)
  }
}

// "todos" | "gostei" | "nao-gostei"
const filtroSelecionado = ref("todos")

const filmesParaExibir = computed(() => {
  switch (filtroSelecionado.value) {
    case "gostei":
      return filmes.value.filter((item) => item.like === true)
    case "nao-gostei":
      return filmes.value.filter((item) => item.like === false)
    default:
      return filmes.value
  }
})

watch(filmes, (novoValor, antigoValor) => {
  localStorage.setItem("vueflix", JSON.stringify(novoValor))
}, { deep: true })

onMounted(() => {
  const dadosLocalStorage = localStorage.getItem("vueflix")
  if (dadosLocalStorage) {
    filmes.value = JSON.parse(dadosLocalStorage)
  }
})
</script>
<template>
  <div class="vueflix">
    <div class="acoes-usuario">
      <div class="filtros">
        <div class="titulo">Filtrar</div>
        <div class="opcoes-filtros">
          <button class="botao" @click="filtroSelecionado = 'todos'"
            :class="{ ativo: filtroSelecionado === 'todos' }">Todos</button>
          <button class="botao" @click="filtroSelecionado = 'gostei'"
            :class="{ ativo: filtroSelecionado === 'gostei' }">Gostei</button>
          <button class="botao" @click="filtroSelecionado = 'nao-gostei'"
            :class="{ ativo: filtroSelecionado === 'nao-gostei' }">Não Gostei</button>
        </div>
      </div>

      <div class="novo-filme">
        <div v-if="adicionandoFilme" class="adicionar-filme">
          <input v-model="nomeFilme" type="text" autocomplete="off" placeholder="Nome do Filme" />
          <input v-model="imagemFilme" type="text" autocomplete="off" placeholder="URL da Imagem" />
          <input v-model="lancamentoFilme" type="text" autocomplete="off" placeholder="Lançamento" />
          <input v-model="generoFilme" type="text" autocomplete="off" placeholder="Gênero" />
          <div class="acoes">
            <button class="botao ativo" @click="salvarFilme">Salvar</button>
            <button class="botao danger ativo" @click="fecharCriacaodeFilme">Cancelar</button>
          </div>
        </div>
        <button v-else class="botao ativo" @click="adicionandoFilme = true">Adicionar Filme</button>
      </div>
    </div>

    <div class="filmes">
      <div v-for="(filme, index) in filmesParaExibir" class="filme">
        <div class="capa-container">
          <div class="acoes-filme">
            <button @click="definirLike(index, true)" class="botao"
              :class="{ ativo: filme.like === true }">Gostei</button>
            <button @click="definirLike(index, false)" class="botao danger" :class="{ ativo: filme.like === false }">Não
              Gostei</button>
            <button @click="excluirFilme(index)" class="botao danger">Excluir</button>
          </div>
          <img class="capa" :src="filme.imagem" alt="" />
        </div>
        <div class="nome">{{ filme.nome }}</div>
        <div class="info">{{ filme.lancamento }} - {{ filme.genero }}</div>
      </div>
      <p v-if="!filmesParaExibir.length" :style="{ flex: 1, textAlign: 'center', marginTop: '16px' }">Não há filmes para
        exibir.</p>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.vueflix {
  padding: 16px;

  .acoes-usuario {
    display: flex;
    justify-content: space-between;
    align-items: end;
    margin: 30px 0;

    .adicionar-filme {
      display: flex;
    }
  }

  .filmes {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
    min-height: 350px;

    .filme {
      .capa-container {
        width: 200px;
        height: auto;
        position: relative;

        .capa {
          width: 100%;
          height: 100%;
        }

        .acoes-filme {
          position: absolute;
          bottom: 0;
          padding-bottom: 12px;
          background-image: linear-gradient(to bottom, rgba(255, 0, 0, 0), rgb(0, 0, 0));
          display: none;
          flex-direction: column;
          width: 100%;
          height: 100%;
          justify-content: flex-end;
        }
      }

      .nome {
        font-weight: bold;
      }

      .info {
        font-size: 12px;
      }

      &:hover {
        .acoes-filme {
          display: flex;
        }
      }
    }
  }
}
</style>
