<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Confirmar Entrega</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: #0f1117;
      color: #e2e8f0;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .card {
      background: #1a1d2e;
      border: 1px solid #2d3148;
      border-radius: 16px;
      padding: 32px 28px;
      width: 100%;
      max-width: 400px;
    }

    .logo {
      display: flex;
      align-items: center;
      gap: 10px;
      margin-bottom: 28px;
    }

    .logo-icon {
      width: 40px;
      height: 40px;
      background: #3b82f6;
      border-radius: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 20px;
    }

    .logo-text {
      font-size: 17px;
      font-weight: 600;
      color: #f1f5f9;
    }

    .logo-sub {
      font-size: 12px;
      color: #64748b;
    }

    label {
      display: block;
      font-size: 13px;
      color: #94a3b8;
      margin-bottom: 6px;
      margin-top: 18px;
    }

    input {
      width: 100%;
      padding: 12px 14px;
      background: #0f1117;
      border: 1px solid #2d3148;
      border-radius: 10px;
      color: #f1f5f9;
      font-size: 16px;
      outline: none;
      transition: border-color 0.2s;
    }

    input:focus {
      border-color: #3b82f6;
    }

    input::placeholder {
      color: #334155;
    }

    button {
      width: 100%;
      padding: 14px;
      margin-top: 24px;
      background: #3b82f6;
      color: #fff;
      border: none;
      border-radius: 10px;
      font-size: 15px;
      font-weight: 600;
      cursor: pointer;
      transition: background 0.2s, transform 0.1s;
    }

    button:hover { background: #2563eb; }
    button:active { transform: scale(0.98); }
    button:disabled { background: #1e3a5f; color: #64748b; cursor: not-allowed; }

    .result {
      margin-top: 20px;
      padding: 14px 16px;
      border-radius: 10px;
      font-size: 14px;
      font-weight: 500;
      display: none;
      line-height: 1.5;
    }

    .result.success {
      background: #052e16;
      border: 1px solid #15803d;
      color: #4ade80;
      display: block;
    }

    .result.error {
      background: #2d0a0a;
      border: 1px solid #991b1b;
      color: #f87171;
      display: block;
    }

    .result-icon {
      font-size: 20px;
      display: block;
      margin-bottom: 6px;
    }

    .spinner {
      display: inline-block;
      width: 16px;
      height: 16px;
      border: 2px solid rgba(255,255,255,0.3);
      border-top-color: #fff;
      border-radius: 50%;
      animation: spin 0.7s linear infinite;
      vertical-align: middle;
      margin-right: 8px;
    }

    @keyframes spin { to { transform: rotate(360deg); } }

    .footer {
      margin-top: 24px;
      text-align: center;
      font-size: 12px;
      color: #334155;
    }
  </style>
</head>
<body>

<div class="card">

  <div class="logo">
    <div class="logo-icon">🛵</div>
    <div>
      <div class="logo-text">Confirmar Entrega</div>
      <div class="logo-sub">Sistema Gerencial</div>
    </div>
  </div>

  <label for="num_pedido">Número do pedido</label>
  <input
    type="number"
    id="num_pedido"
    placeholder="Ex: 1042"
    inputmode="numeric"
    autocomplete="off"
  />

  <label for="codigo">Código informado pelo cliente</label>
  <input
    type="text"
    id="codigo"
    placeholder="Ex: AB12"
    autocomplete="off"
    style="text-transform: uppercase; letter-spacing: 2px;"
  />

  <button id="btn-confirmar" onclick="confirmarEntrega()">
    Confirmar entrega
  </button>

  <div class="result" id="resultado"></div>

  <div class="footer">Digite o número do pedido e o código que o cliente te informou</div>

</div>

<script>
  // =============================================
  // CONFIGURAÇÃO — substitua pelos seus valores
  // =============================================
  const SUPABASE_URL = "https://oaxrdgmrzllyhjiskmfj.supabase.co";
  const SUPABASE_ANON_KEY = "sb_publishable_DbRmua1Lzy9AFcAXBPYc6w_Nws3TnXm";
  // =============================================

  const resultEl = document.getElementById("resultado");
  const btn = document.getElementById("btn-confirmar");

  async function confirmarEntrega() {
    const numPedido = document.getElementById("num_pedido").value.trim();
    const codigo = document.getElementById("codigo").value.trim().toUpperCase();

    resultEl.className = "result";
    resultEl.style.display = "none";

    if (!numPedido || !codigo) {
      mostrarErro("Preencha o número do pedido e o código.");
      return;
    }

    btn.disabled = true;
    btn.innerHTML = '<span class="spinner"></span>Verificando...';

    try {
      // 1. Busca o pedido pelo id
      const busca = await fetch(
        `${SUPABASE_URL}/rest/v1/entregas?id=eq.${numPedido}&select=id,codigo_validacao,status`,
        {
          headers: {
            apikey: SUPABASE_ANON_KEY,
            Authorization: `Bearer ${SUPABASE_ANON_KEY}`,
          },
        }
      );

      const pedidos = await busca.json();

      if (!pedidos || pedidos.length === 0) {
        mostrarErro("Pedido não encontrado. Verifique o número.");
        return;
      }

      const pedido = pedidos[0];

      if (pedido.status === "Entregue") {
        mostrarErro("Este pedido já foi confirmado anteriormente.");
        return;
      }

      if (pedido.codigo_validacao !== codigo) {
        mostrarErro("Código incorreto. Peça ao cliente para confirmar o código.");
        return;
      }

      // 2. Atualiza status para Entregue
      const update = await fetch(
        `${SUPABASE_URL}/rest/v1/entregas?id=eq.${numPedido}`,
        {
          method: "PATCH",
          headers: {
            apikey: SUPABASE_ANON_KEY,
            Authorization: `Bearer ${SUPABASE_ANON_KEY}`,
            "Content-Type": "application/json",
            Prefer: "return=minimal",
          },
          body: JSON.stringify({ status: "Entregue" }),
        }
      );

      if (!update.ok) {
        mostrarErro("Erro ao confirmar. Tente novamente.");
        return;
      }

      mostrarSucesso(`Pedido #${numPedido} confirmado com sucesso!\nO sistema do estabelecimento foi atualizado.`);
      document.getElementById("num_pedido").value = "";
      document.getElementById("codigo").value = "";

    } catch (e) {
      mostrarErro("Sem conexão. Verifique sua internet e tente novamente.");
    } finally {
      btn.disabled = false;
      btn.innerHTML = "Confirmar entrega";
    }
  }

  function mostrarSucesso(msg) {
    resultEl.innerHTML = `<span class="result-icon">✓</span>${msg.replace(/\n/g, "<br>")}`;
    resultEl.className = "result success";
  }

  function mostrarErro(msg) {
    resultEl.innerHTML = `<span class="result-icon">✕</span>${msg}`;
    resultEl.className = "result error";
  }

  // Confirmar com Enter
  document.addEventListener("keydown", (e) => {
    if (e.key === "Enter") confirmarEntrega();
  });
</script>

</body>
</html>
