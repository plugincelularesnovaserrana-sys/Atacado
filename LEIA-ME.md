# Atacado com reserva de estoque e gestão de pedidos

## Como funciona

- O **produtos.json** continua sendo o estoque "físico" (quantidade `qtd` de cada aparelho). Você segue editando só ele (pelo admin.html ou direto no repositório).
- Quando um cliente envia o pedido, o site **reserva** os aparelhos num pequeno banco em tempo real (Firebase). O estoque que aparece para todo mundo é `qtd` do JSON menos o que está reservado.
- Dois clientes pedindo o mesmo item ao mesmo tempo: o primeiro que chegar leva; o segundo recebe o alerta "Item indisponível" e o carrinho dele é ajustado. A reserva é "tudo ou nada": se um item do carrinho falhar, nenhum é reservado.
- A página **pedidos.html** mostra os pedidos ao vivo (com som quando chega um novo). Ali você confirma, altera quantidades, remove itens, cancela, reabre e conclui.
  - Alterar ou cancelar um pedido aberto: devolve a reserva na hora.
  - Concluir: baixa a quantidade no produtos.json (precisa do token do GitHub, o mesmo do admin).
  - Cancelar um pedido já concluído: devolve a quantidade no produtos.json.
- O site se atualiza sozinho: as reservas chegam em tempo real e o produtos.json é conferido a cada 45 segundos (e quando o cliente volta para a aba). Só o JSON precisa ser mexido, os HTML não.
- O cliente ganha o botão **Meus pedidos**, onde vê o status mudar sozinho.

## Configurar o Firebase (uma vez só, gratuito, sem cartão)

1. Entre em https://console.firebase.google.com, **Adicionar projeto** (pode desligar o Google Analytics).
2. **Build → Realtime Database → Criar banco de dados**. Escolha qualquer região e o modo **bloqueado**. Copie o endereço que aparece no topo (termina com `firebaseio.com`).
3. Aba **Regras**: apague tudo, cole o conteúdo do arquivo **firebase-regras.json** e clique em **Publicar**.
4. **Build → Authentication → Vamos começar → E-mail/senha → Ativar**. Na aba **Usuários**, **Adicionar usuário** com o e-mail e a senha que você vai usar para entrar no pedidos.html.
5. Ainda em Authentication, aba **Configurações → Domínios autorizados → Adicionar domínio**: `plugincelularesnovaserrana-sys.github.io`.
6. **Configurações do projeto (engrenagem) → Seus aplicativos → ícone `</>` (Web)**. Registre o app e copie os dados (`apiKey`, `authDomain`, `projectId`, `appId`).
7. Abra o **config.json** e cole esses dados, mais o `databaseURL` do passo 2.
8. Suba para o repositório: **index.html, pedidos.html, admin.html, config.json** (o firebase-regras.json e este arquivo são só para consulta).

Depois disso:
- Abra `.../Atacado/pedidos.html`, entre com o e-mail e a senha do passo 4.
- Abra **Estoque no GitHub** e cole o token (o mesmo do admin). Marque "Lembrar" se quiser.

## Pontos de atenção

- **Cadastro com quantidade**: só itens com `qtd` entram no controle de reserva. Item antigo sem quantidade (novos cadastrados antes) continua com estoque livre e não é reservado. Salve a lista uma vez no admin para todo item receber código e quantidade.
- **Admin e pedidos mexem no mesmo JSON**: o admin agora recusa o envio se o produtos.json mudou depois que você o carregou (por exemplo, um pedido foi concluído). Nada é sobrescrito: toque em "Salvar e carregar lista" e refaça a alteração.
- **Vendeu na loja física**: continue dando −1 no admin. Isso mexe na quantidade física; as reservas dos pedidos abertos seguem separadas.
- **Aviso amarelo "Reservas fora de sincronia"** na página de pedidos: aparece se alguma reserva não bate com os pedidos abertos (por exemplo, queda de internet no meio de um envio). O botão **Recalcular reservas** corrige.
- **Segurança**: o site público precisa gravar reservas e pedidos, então alguém mal-intencionado com conhecimento técnico poderia mexer nelas. Como o pedido sempre passa pela sua confirmação, o risco é pequeno e o "Recalcular reservas" desfaz. Se um dia precisar de proteção total, o caminho é mover a reserva para uma Cloud Function (exige plano pago do Firebase).
- Sem o config.json preenchido, o site continua funcionando como antes (pedido só pelo WhatsApp, sem reserva).
