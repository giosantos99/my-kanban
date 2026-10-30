<template>
<q-page v-if="!loading" class="q-pa-sm">

  <div class="container row no-wrap scroll">
    <template v-if="paineis?.length">
      <div
        v-for="painel in paineis"
        :key="painel.id"
        :style="{ order: painel.ordenacao_painel }"
        :class="[
          'col-xs-12 col-md-4 q-px-sm text-white q-pb-sm',
          `painel-${painel.id}`
        ]"
      >
        <q-card class="fit painel bg-accent">
          <q-card-section
            class="row justify-between items-center q-pa-xs bg-secondary"
            style="min-height: 42px"
          >

            <div class="row items-center no-wrap">
              <q-icon :name="painel.icone" class="q-mr-sm" />
              <span class="text-weight-medium">{{ painel.status }}</span>
            </div>

            <q-btn
              v-if="tarefas[painel.id]?.length"
              icon="delete_sweep"
              round
              dense
              flat
              @click="excluirTarefasDoPainel(painel)"
            >
              <q-tooltip v-if="!$q.screen.lt.md">Limpar tudo</q-tooltip>
            </q-btn>
          </q-card-section>

          <div class="container-painel column no-wrap scroll text-dark">

            <div
              v-if="!tarefas[painel.id]?.length"
              class="flex flex-center items-center q-gutter-x-sm text-grey-7 q-pt-xl"
            >
              <q-icon name="warning" size="sm" color="warning" />
              <span>Nenhuma tarefa adicionada.</span>
            </div>

            <LayoutCardsWeb
              v-if="!$q.screen.lt.md"

              :_tarefas="tarefas"
              :painelId="painel.id"

              @excluir-tarefa="idTarefa => excluirTarefa(painel.id, idTarefa)"
              @editar-tarefa="tarefa => editarTarefa(painel.id, tarefa)"
              @drag-drop="({ evt, tarefas }) => onDragDrop(evt, tarefas, painel.id)"
            />

            <!-- LAYOUT MOBILE -->
            <CardTarefa
              v-else
              v-for="(tarefa, index) in tarefas[painel.id]"
              :key="tarefa.id"

              :tarefa="tarefa"
              :exibirSetaCima="!!index"
              :exibirSetaBaixo="index !== tarefas[painel.id].length - 1"

              :style="{ order: tarefa.ordenacao_tarefa }"
              class="q-ma-md"

              @alterar-painel="novoPainel => alterarPainelMobile(novoPainel, painel.id, tarefa)"
              @editar-tarefa="tarefa => editarTarefa(painel.id, tarefa)"
              @excluir-tarefa="idTarefa => excluirTarefa(painel.id, idTarefa)"
              @mover-tarefa="action => moverTarefaMobile(action, painel.id, tarefa, index)"
            />
          </div>
        </q-card>
      </div>
    </template>
  </div>

  <img
    v-if="mostrarComemoracao"
    src="@/assets/done.gif"
    class="done"
  />

  <ButtonNovaTarefa @adicionar-tarefa="modalAdicionarTarefa"/>

</q-page>

<q-page v-else class="q-pa-sm">
  <div class="container row no-wrap scroll">
    <template v-for="n in 3" :key="n">
      <div class="col-xs-12 col-md-4 q-px-sm text-white q-pb-sm">
        <q-skeleton square class="fit painel" />
      </div>
    </template>
  </div>

</q-page>
</template>

<script>
import { defineAsyncComponent } from 'vue'

import { ButtonNovaTarefa, CardTarefa } from '@/components/UI'
import LayoutCardsWeb from '@/components/LayoutCardsWeb.vue'

export default {
  components: {
    ButtonNovaTarefa,
    CardTarefa,
    LayoutCardsWeb
  },

  data () {
    return {
      loading: false,
      mostrarComemoracao: false,
      dadosCarregados: false,
      paineis: [
        {
          status: 'Fazer',
          id: 'fazer',
          icone: 'play_circle',
          ordenacao_painel: 1
        },
        {
          status: 'Fazendo',
          id: 'fazendo',
          icone: 'fa-solid fa-spinner',
          ordenacao_painel: 2
        },
        {
          status: 'Feito',
          id: 'feito',
          icone: 'check_circle',
          ordenacao_painel: 3
        }
      ],
      tarefas: {
        fazer: [],
        fazendo: [],
        feito: []
      },
    }
  },

  methods: {
    normalizarPainel (status) {
      const lista = this.tarefas[status] || []

      this.tarefas[status] = lista.map((tarefa, index) => ({
        ...tarefa,
        status,
        ordenacao_tarefa: index
      }))
    },

    verificarComemoracao () {
      const { fazer, fazendo, feito } = this.tarefas

      if (!fazer.length && !fazendo.length && feito.length > 0) {
        this.mostrarComemoracao = true

        setTimeout(() => {
          this.mostrarComemoracao = false
        }, 1600)
      }
    },

    moverTarefaMobile (action, painel, tarefa, index) {
      const destino = action === 'arrow_drop_up'
        ? index - 1
        : index + 1

      const lista = this.tarefas[painel]

      if (destino < 0 || destino >= lista.length) return

      const [tarefaMovida] = lista.splice(index, 1)

      lista.splice(destino, 0, tarefaMovida)

      this.onDragDrop(
        { moved: { element: tarefaMovida } },
        this.tarefas,
        painel
      )
    },

    alterarPainelMobile (novoPainel, painelAtual, tarefa) {
      if (novoPainel === painelAtual) return

      this.tarefas[novoPainel].push({ ...tarefa })

      this.tarefas[painelAtual] = this.tarefas[painelAtual].filter(
        item => item.id !== tarefa.id
      )

      this.onDragDrop(
        {
          added: {
            element: { ...tarefa }
          }
        },
        this.tarefas,
        novoPainel
      )

      this.onDragDrop(
        {},
        this.tarefas,
        painelAtual
      )

      const containerPainel = document.querySelector(
        `.painel-${novoPainel}`
      )

      containerPainel?.scrollIntoView({
        behavior: 'smooth'
      })
    },

    onDragDrop (evt = {}, _tarefas, status) {
      this.tarefas = _tarefas

      const { added } = evt

      if (added?.element) {
        added.element.status = status
      }

      this.normalizarPainel(status)

      if (added?.element && status === 'feito') this.verificarComemoracao()
    },

    excluirTarefasDoPainel ({ status, id }) {
      const dialog = this.$q.dialog({
        component: defineAsyncComponent(
          () => import('@/components/Modal/ModalNotificacao.vue')
        ),
        componentProps: {
          texto: `Tem certeza que deseja excluir todas as tarefas do painel "${status}" ?`,
          titulo: 'Excluir Tarefas'
        }
      })

      dialog.onOk(() => {
        const quantidade = this.tarefas[id].length

        if (!quantidade) return

        this.tarefas[id] = []

        this.$q.notify({
          color: 'positive',
          message: `${quantidade} tarefa(s) excluída(s) com sucesso`
        })
      })
    },

    editarTarefa (painel, tarefa) {
      const index = this.tarefas[painel].findIndex(
        item => item.id === tarefa.id
      )

      if (index === -1) return

      const dt_criacao = new Intl.DateTimeFormat(
        'pt-BR',
        { dateStyle: 'short' }
      ).format(new Date())

      this.tarefas[painel].splice(index, 1, {
        ...tarefa,
        status: painel,
        dt_criacao
      })

      this.$q.notify({
        color: 'positive',
        message: 'Tarefa editada com sucesso'
      })
    },

    excluirTarefa (painel, id, notificacao = true) {
      const quantidadeAnterior = this.tarefas[painel].length

      this.tarefas[painel] = this.tarefas[painel].filter(
        tarefa => tarefa.id !== id
      )

      if (this.tarefas[painel].length === quantidadeAnterior) return

      this.normalizarPainel(painel)

      if (notificacao) {
        this.$q.notify({
          color: 'positive',
          message: 'Tarefa excluída com sucesso'
        })
      }
    },

    fnAdicionarTarefa (status, descricao, notificacao = true) {
      const lista = this.tarefas[status]

      const ordenacao_tarefa = lista.length
        ? Math.max(
            ...lista.map(tarefa => tarefa.ordenacao_tarefa)
          ) + 1
        : 0

      const dt_criacao = new Intl.DateTimeFormat(
        'pt-BR',
        { dateStyle: 'short' }
      ).format(new Date())

      const id = globalThis.crypto?.randomUUID
        ? globalThis.crypto.randomUUID()
        : `${Date.now()}-${Math.random().toString(16).slice(2)}`

      const novaTarefa = {
        id,
        status,
        descricao,
        ordenacao_tarefa,
        dt_criacao
      }

      lista.push(novaTarefa)

      if (notificacao) {
        this.$q.notify({
          color: 'positive',
          message: 'Tarefa adicionada com sucesso'
        })
      }

      return novaTarefa
    },

    modalAdicionarTarefa () {
      const dialog = this.$q.dialog({
        component: defineAsyncComponent(
          () => import('@/components/Modal/ModalForm.vue')
        )
      })

      dialog.onOk(({ tarefa }) => {
        this.fnAdicionarTarefa('fazer', tarefa)

        if (this.$q.screen.lt.md) {
          const painelFazer = document.querySelector('.painel-fazer')

          painelFazer?.scrollIntoView({
            behavior: 'smooth'
          })
        }
      })
    },

    fetchTarefas () {
      try {
        this.loading = true

        const dadosSalvos = localStorage.getItem('mykanban_tarefas')

        if (dadosSalvos) {
          const dados = JSON.parse(dadosSalvos)

          this.tarefas = {
            fazer: Array.isArray(dados.fazer) ? dados.fazer : [],
            fazendo: Array.isArray(dados.fazendo) ? dados.fazendo : [],
            feito: Array.isArray(dados.feito) ? dados.feito : []
          }
        }
      } catch {
        this.$q.notify({
          color: 'negative',
          message: 'Erro ao carregar as tarefas salvas'
        })
      } finally {
        this.dadosCarregados = true
        this.loading = false
      }
    }
  },


  watch: {
    tarefas: {
      deep: true,

      handler (novasTarefas) {
        if (!this.dadosCarregados) return

        try {
          localStorage.setItem(
            'mykanban_tarefas',
            JSON.stringify(novasTarefas)
          )
        } catch (error) {
          console.error('Erro ao salvar tarefas:', error)

          this.$q.notify({
            color: 'negative',
            message: 'Não foi possível salvar as tarefas no navegador'
          })
        }
      }
    }
  },

  mounted () {
    this.fetchTarefas()
  }
}
</script>

<style lang="sass" scoped>

.container
  min-height: calc( 100vh - 66px )
  max-height: calc( 100vh - 66px )

  &-painel
    max-height: calc( 100vh - 120px )

.done
  position: fixed
  top: 50%
  left: 50%
  transform: translate(-50%, -50%)
  z-index: 999
  pointer-events: none
  max-width: 300px
  max-height: 300px

</style>
