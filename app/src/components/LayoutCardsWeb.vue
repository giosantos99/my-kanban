<template>
  <div>
    <q-card-section>
      <draggable
        :list="_tarefas[painelId]"
        group="tarefas"
        item-key="id"
        @change="mudarPosicaoTarefa"
        style="min-height: 150px"
      >
        <template #item="{ element }">
          <CardTarefa
            :tarefa="element"
            :style="{ order: element.ordenacao_tarefa }"
            class="q-mb-md"
            @excluir-tarefa="(idTarefa) => $emit('excluir-tarefa', idTarefa)"
            @editar-tarefa="(tarefa) => $emit('editar-tarefa', tarefa)"
          />
        </template>
      </draggable>
    </q-card-section>

    <q-card-section
      v-if="!$q.screen.lt.md && _tarefas[painelId]?.length && !Boolean(moverTarefa)"
      class="column items-center"
      style="margin-top: 120px"
    >
      <q-icon name="sym_o_drag_pan" size="md" color="grey-8" />

      <div class="text-center text-grey-7 q-pt-sm">Arraste para mover</div>
    </q-card-section>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import { CardTarefa } from './UI'

export default {
  name: 'LayoutCardsWeb',

  props: {
    _tarefas: {
      type: Object,
      required: true,
      default: () => ({}),
    },

    painelId: {
      type: String,
      required: true,
    },
  },

  emits: ['excluir-tarefa', 'editar-tarefa', 'drag-drop'],

  components: {
    draggable,
    CardTarefa,
  },

  data() {
    return {
      moverTarefa: localStorage.getItem('moverTarefa') || 0,
    }
  },

  methods: {
    mudarPosicaoTarefa(evt) {
      localStorage.setItem('moverTarefa', 1)
      this.moverTarefa = 1

      this.$emit('drag-drop', {
        evt,
        tarefas: this._tarefas,
      })
    },
  },
}
</script>
