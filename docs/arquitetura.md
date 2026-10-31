# Arquitetura

## Visão geral

Dois produtos independentes ligados por uma API pequena: o **app Android**, que decide e mostra o veredito de cada oferta no momento em que ela aparece, e o **painel web**, que só recebe corridas já aceitas e as transforma em indicadores.

## App: leitura em três responsabilidades

O problema que mais custou foi fazer a leitura funcionar de forma confiável. A solução separou o que antes estava misturado num único fluxo dirigido por eventos.

1. **Visibilidade.** O app de corrida está na tela agora? Olha todas as janelas de aplicação visíveis, não só o pacote do último evento. Uma pequena histerese (2,5 s) evita "piscadas" quando a lista de janelas vem vazia por um instante durante transições. Janelas de sistema, como a bolha flutuante dos apps de corrida, não contam como "app visível".
2. **Política de captura.** Precisa e pode capturar agora? Respeita pausa do usuário, tela apagada e bloqueada. Trata cada tipo de falha de `takeScreenshot` com um recuo próprio (intervalo curto demais, janela protegida, sem permissão, erro interno). Garante uma única captura em andamento e descarta, de forma lógica, resultados que ficaram obsoletos porque o contexto mudou.
3. **Captura, OCR e interpretação.** Um laço único, com intervalo ajustável (0,5, 1 ou 2 s), aciona captura, OCR e parser mesmo sem nenhum evento de acessibilidade. Os eventos só servem para antecipar a próxima leitura.

O OCR também lê o próprio overlay do app. Um filtro descarta as linhas que caem dentro das áreas do overlay, para que o veredito nunca vire "dado da oferta".

## App: interpretação das ofertas

- **Parser por app e por tela.** Cartão de uma oferta, lista com várias ofertas, tela da Uber.
- **Conferência por redundância.** Como o OCR confunde caracteres (`$` com `S`, `0` com `O`) e embaralha a ordem das linhas, o parser usa o que a própria tela mostra duas vezes: os dois trechos precisam somar o total exibido, e o preço dividido pelos km precisa bater com o R$/km exibido. Só vira oferta o cartão em que as conferências fecham.
- **Motor de lucro** puro, sem Android: R$/km e R$/hora brutos (a corrida paga buscar e levar), nota de 0 a 10 ponderada, lucro estimado por custo por km.

## Envio

Cada corrida aceita entra numa fila local (arquivo de linhas) e é enviada por HTTPS com uma chave própria do aparelho. O servidor guarda só o hash da chave e trata a chegada como idempotente por `(aparelho, id do cliente)`, então reenvios não duplicam.

## Painel

- Next.js com server components: as consultas rodam no servidor, sem API intermediária para o painel.
- Login por senha única, com cookie `httpOnly` assinado.
- Gráficos em SVG renderizados no servidor, sem biblioteca.
- Classificação de cada corrida (boa, média ou ruim) com as mesmas metas do app.
- Meta do mês: contas cadastradas, lucro estimado acumulado, ritmo diário necessário e projeção; gravação das contas em transação.

## Dados

Tabelas `devices` (aparelhos e hash da chave), `rides` (plataforma, horário, valor, km e minutos de busca e de viagem, nota, lucro estimado) e `goal_items` (contas da meta mensal).
