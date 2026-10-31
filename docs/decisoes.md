# Decisões e problemas reais

## Por que OCR, e não a árvore de acessibilidade
Nos apps de corrida, a oferta é desenhada em um motor próprio (não usa componentes nativos do Android). A árvore de acessibilidade não traz valores nem distâncias, então a única fonte é a imagem da tela. O OCR roda no aparelho (ML Kit), sem enviar imagens a lugar nenhum.

## Por que leitura periódica, e não só eventos
A oferta de uma das plataformas aparece sem gerar eventos de acessibilidade. Um app que só reage a eventos simplesmente não vê essas ofertas. A leitura periódica controlada (com um único laço, uma captura por vez, pausa e recuo por falha) resolve isso sem gastar bateria quando o app de corrida não está na tela.

## "Compilar" não é "funcionar"
A validação exige uma oferta real chegando sem evento. O diagnóstico local conta ofertas lidas, falhas de captura, resultados descartados e eventos recebidos: se "ofertas lidas" sobe enquanto "eventos" fica parado, a leitura periódica está de fato fazendo o trabalho. Esses contadores não guardam nenhum texto de tela.

## Bugs que ensinaram algo
- **A lista de ofertas não era entendida.** Os cartões terminam em botões diferentes ("Aceitar por" quando há negociação, "Escolher" quando não há). O parser só fechava o cartão no primeiro caso, juntava os trechos de duas ofertas e descartava tudo. Passou a fechar também pelo outro botão e pelo preço do cartão seguinte.
- **O app lia um print.** A bolha flutuante do app de corrida, que é uma janela de sistema, era contada como "app visível", então um print guardado no WhatsApp gerava veredito. Passou a valer só janela de aplicação.
- **Overlay que não sumia, overlay que sumia cedo.** Os dois vinham do mesmo ponto: decidir esconder pela ausência de leitura. Virou um temporizador com dois prazos (nada que pareça oferta; algo que parece oferta mas não foi entendido).
- **Piscadas de visibilidade.** A lista de janelas vem vazia por instantes durante transições. Uma histerese curta, que só vale se nenhum outro app assumiu a tela, estabilizou a leitura.

## Privacidade como requisito de projeto
Só números e o nome da plataforma saem do celular. O modo de captura, que grava texto de tela para estudar novos formatos, é opt-in, avisa que o log pode ter dados pessoais e fica desligado por padrão. O diagnóstico padrão registra apenas códigos e contadores.

## Dados, não planilha
O histórico começou numa planilha. Virou um painel próprio por três motivos: filtros e gráficos que a planilha não dá, a meta do mês calculada contra o lucro estimado, e um caminho para o app enviar sozinho cada corrida.

## O que ainda não faz
- O painel só tem corridas **aceitas**; ofertas recusadas não são enviadas (decisão de privacidade e simplicidade, a rever).
- O custo por km é uma estimativa até haver consumo e preço de combustível reais configurados.
- Cada plataforma pode mudar o layout da tela; os parsers têm testes com leituras reais, mas dependem de manutenção.
