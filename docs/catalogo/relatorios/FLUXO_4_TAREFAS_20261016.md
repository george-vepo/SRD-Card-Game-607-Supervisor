# Fluxo de execução — SRD Card Game (4 tarefas / 60 minutos)

**Interpretação:** quatro jobs one-shot lançados em T+00, T+15, T+30 e T+45. Total de uma hora para os disparos; a conclusão pode exceder a janela.

## Arquivos

- `docs/catalogo/relatorios/fila-imagens-1320-20261016.csv`: **1.320** linhas; 1.284 com `PENDENTE=SIM`, 36 já geradas (21 no manifesto da branch principal e 15 no pacote local, ainda não sincronizadas).
- `docs/catalogo/relatorios/fila-fichas-1320-20261016.csv`: **1.320** linhas; 1.212 com `PENDENTE=SIM`, 108 com texto integral arquivado.
- Todo CSV usa **UTF-8 com BOM e `;`**, uma linha por criatura, mesma `CHAVE` e `ORDEM_1320`.
- Nunca usar a linha do CSV de fichas como "próxima" para imagem: selecionar pela fila de imagens e, então, procurar a mesma CHAVE na fila de fichas.

## Máquina de estados

`PENDENTE` contém **SIM** ou **NAO**, indicando se falta concluir aquela entrega específica. `STATUS` assume:
- `PENDENTE`: nunca processado ou liberado para retry;
- `EM_PROCESSAMENTO`: reserva exclusiva do job atual;
- `CONCLUIDO`: entrega comprovada e persistida;
- `AGUARDANDO_FONTE`, `REVISAO_DIREITOS`, `BLOQUEADO_GERADOR`, `ERRO_ZIP` e `ERRO_UPLOAD`: pendente, com motivo específico; pode ser reprogramado após a causa ser resolvida.

Consistência: `CONCLUIDO` exige `PENDENTE=NAO`; demais situações ainda em aberto mantêm `PENDENTE=SIM`. Uma ficha já pronta pode ficar `CONCLUIDO` quando a imagem da mesma criatura estiver pendente.

## Fases de um job

1. **Ler e validar** → conferir 1.320 CHAVES, unicidade, origem no manifesto e duas tabelas.
2. **Selecionar** → menor ORDEM com imagem `PENDENTE=SIM`, `STATUS=PENDENTE`.
3. **Reservar atomica e remotamente** → mudar `STATUS=EM_PROCESSAMENTO` nas duas filas quando aplicável; gerar identificador único; commit otimista/lease. Em disputa, recarregar.
4. **Pesquisar criatura** → identidade, livro, edição, anatomia, permissões.
5. **Gerar arte** → prompt conforme padrão 2D dark high fantasy 2:3; validar enquadramento/anatomia; exportar WebP.
6. **Produzir ficha** → pesquisar a mesma espécie; aproveitar documento integral quando disponível ou estruturar original com referências sem inventar números. Caso incompleta, deixar pendente.
7. **Montar ZIP** → imagem, prompt, ficha (se completa), fontes, hashes e manifesto; path `operational/outputs/geracao-criaturas/<data>/<chave>-<nome>-<utc>.zip`.
8. **Reabrir, testar e armazenar ZIP** → verificar integridade, conteúdo e persistência real.
9. **Atualizar filas** → cada pendência separadamente com evidência; liberar/expirar reservas somente com política de retry e logs de auditoria.
10. **Reportar execução** → links/paths, estado imagem/ficha, erros e próximo item elegível.

## Agenda

| Tarefa | Disparo relativo | Execução | Seleção |
| --- | --- | --- | --- |
| JOB-01 | T+00 | uma espécie | próxima livre |
| JOB-02 | T+15 | uma espécie | próxima livre, excluindo reservas |
| JOB-03 | T+30 | uma espécie | próxima livre, excluindo reservas |
| JOB-04 | T+45 | uma espécie | próxima livre, excluindo reservas |

**Primeiras candidatas** da fila de imagens na versão-base, caso não haja intervenção entre tarefas: Neogi Hatchling, Derro, Star Spawn Grue e Dolgrim. Como a seleção é feita em tempo real, esses nomes são apenas previsão, não atribuição rígida.

## Critérios de aceite

- ZIP contém `imagem.webp` realmente existente e válida. Prompts isolados ou URLs não comprovam arte concluída.
- Ficha tem dados comprovados, edição rastreável e licença/limites documentados. Não reproduzir texto editorial protegido de terceiros.
- Updates são atômicos, sem duas tarefas com a mesma chave; se um upload falhar, registrar pendência.
- O catálogo publicado não é atualizado automaticamente por criar esses CSVs ou ZIPs; atualização visual depende de processo de curadoria e publicação independente.
- ChatGPT Tasks recorrentes têm limite de **uma execução por hora**. Para quatro disparos espaçados em 15 minutos, use um agendador externo compatível, ou quatro jobs one-shot **somente se** a plataforma explicitamente permitir essa configuração. Os prompts não constituem agendamento.
