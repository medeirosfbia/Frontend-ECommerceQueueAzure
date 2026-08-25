<template>
  <div class="checkout-card">
    <h2>Confirmação de Venda (E-commerce)</h2>
    
    <form @submit.prevent="confirmarVenda">
      <div class="field">
        <label>Cliente:</label>
        <input v-model="form.cliente" type="text" required placeholder="Ex: Maria Silva" />
      </div>

      <div class="field">
        <label>Valor Total (R$):</label>
        <input v-model.number="form.valorTotal" type="number" step="0.01" required />
      </div>

      <div class="field">
        <label>Itens do Pedido:</label>
        <input v-model="form.itensTexto" type="text" placeholder="Ex: Camiseta Verde, Calça Jeans" />
      </div>

      <button type="submit" :disabled="processando">
        {{ processando ? 'Confirmando...' : 'Confirmar Venda' }}
      </button>
    </form>

    <p v-if="mensagem" :class="tipoMensagem" class="status-msg">{{ mensagem }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const API_URL = 'http://localhost:3000';

const form = ref({
  cliente: '',
  valorTotal: 0,
  itensTexto: ''
});

const processando = ref(false);
const mensagem = ref('');
const tipoMensagem = ref('sucesso');

async function confirmarVenda() {
  processando.value = true;
  mensagem.value = '';

  try {
    const payload = {
      idVenda: `PEDIDO_${Date.now()}`,
      cliente: form.value.cliente,
      valorTotal: form.value.valorTotal,
      itens: form.value.itensTexto.split(',').map(item => item.trim())
    };

    const response = await fetch(`${API_URL}/api/vendas/confirmar`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    const data = await response.json();

    if (!response.ok) throw new Error(data.message || 'Erro ao confirmar venda');

    mensagem.value = `Sucesso! Mensagem enviada para a fila Azure (MessageID: ${data.messageId})`;
    tipoMensagem.value = 'sucesso';

    // Limpa formulário
    form.value.cliente = '';
    form.value.valorTotal = 0;
    form.value.itensTexto = '';

  } catch (err) {
    mensagem.value = err.message;
    tipoMensagem.value = 'erro';
  } finally {
    processando.value = false;
  }
}
</script>