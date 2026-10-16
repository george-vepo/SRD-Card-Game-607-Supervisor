# Prompt-mestre — 4 execuções em 1 hora, uma a cada 15 minutos

Você é um agente de produção de artes e fichas do projeto SRD Card Game.

## Contexto e arquivos de controle
Repositório: `george-vepo/SRD-Card-Game-607-Supervisor`.
- Fila de imagens: `docs/catalogo/relatorios/fila-imagens-1320-20261016.csv`.
- Fila de fichas: `docs/catalogo/relatorios/fila-fichas-1320-20261016.csv`.
- Fonte canônica de ordem/identidade: `docs/catalogo/data/megalista-1320.json`.
- Fichas locais: `docs/catalogo/data/fichas-607.json`.
- Manifesto de artes já publicadas: `docs/catalogo/data/artes-geradas-20261015.json`.
- Pacote adicional de 15 artes geradas (ainda não publicadas): `srd-catalog-artes-20261016.zip` (não presumir que está disponível; confirmar acesso).
- Procedimento detalhado: `docs/catalogo/relatorios/FLUXO_4_TAREFAS_20261016.md`.

## Agenda solicitada
Crie/executa **quatro jobs independentes e únicos** durante uma janela de 60 minutos, iniciando em **T+0, T+15, T+30 e T+45 minutos** a partir do início definido pelo operador. Cada job processa **no máximo uma criatura**. Não interprete isso como quatro jobs de duração de uma hora. Quando uma plataforma não suporta jobs a cada 15 minutos, não alegue que foram agendados: proporcione um orquestrador que suporte a cadência ou obtenha aprovação para outra frequência.

## Algoritmo obrigatório de cada job
1. Ler a versão atual dos **dois** CSVs em UTF-8 com BOM e separador `;`; confirmar 1.320 registros únicos, mesma `CHAVE` e mesma `ORDEM_1320`.
2. Escolher o registro de menor `ORDEM_1320` em fila de imagens onde `PENDENTE=SIM` e `STATUS=PENDENTE` (não escolher as marcadas `CONCLUIDO`, `EM_PROCESSAMENTO` ou `BLOQUEADO`).
3. Reservar a `CHAVE` **atomicamente** antes de gerar qualquer arquivo: atualizar ambos os CSVs com `STATUS=EM_PROCESSAMENTO` para a mesma `CHAVE` quando estiverem pendentes, `RESERVADO_POR=<job-id>` e horário UTC. Usar commit com verificação da revisão atual (compare-and-swap ou fast-forward); em conflito, recarregar CSVs e selecionar outra criatura. Nunca processar a mesma chave em paralelo.
4. Pesquisar descrição e anatomia com fontes confiáveis (preferencialmente documentação do projeto, editoras e compêndios legais). Não presumir que uma criatura semelhante seja a mesma espécie; preservar o nome e a edição da megalista.
5. Criar **uma ilustração nova, autoral, 2D**, vertical 2:3 (preferencialmente 1024 × 1536 px), pintura manual digital `premium hand-painted dark high fantasy trading card / classical RPG bestiary`: figura inteira, anatomia condizente, identidade visual e detalhes únicos, composição tridimensional convincente sem aspecto CGI, paisagem atmosférica coerente, luz cinematográfica, bom contraste, sem texto, moldura, watermark, armas tecnológicas injustificadas ou membros cortados. Adaptar cenário/paleta à criatura. Usar gerador de imagens realmente disponível; caso não haja ferramenta, registrar `BLOQUEADO_GERADOR`, **não** substituir por link, prompt ou imagem de outra criatura.
6. Validar visualmente espécie/forma/quantidade de membros e imagem completa; exportar `imagem.webp` em 2:3 e manter `prompt-imagem.md` com o prompt integral e parâmetros. Se a arte falhar, registrar falha, não concluir o item.
7. Para a **mesma CHAVE**, consultar a fila de fichas. Se `PENDENTE=NAO`, reutilizar o registro **já integral** da origem permitida e não refazer mecanicamente; preservar proveniência. Se `PENDENTE=SIM`, pesquisar a fonte correspondente à espécie/edição; compilar uma **ficha estruturada, original e verificável** (`ficha.json` e `ficha.md`) com campos verificáveis: nome, fonte, edição, porte, tipo, alinhamento se publicado, ND, CA, PV, deslocamento, atributos, salvaguardas, perícias, resistências/imunidades, sentidos, idiomas, traços, ações, ações bônus, reações, lendárias/conjuração quando aplicáveis, origens e lacunas. Não inventar números nem copiar integralmente textos editoriais protegidos. Quando direitos ou acesso impedirem uma ficha completa, criar apenas `pesquisa-ficha.md`, marcar `AGUARDANDO_FONTE` ou `REVISAO_DIREITOS`, manter `PENDENTE=SIM` e declarar campos ausentes.
8. Empacotar em ZIP **por criatura**: `imagem.webp`, `prompt-imagem.md`, `ficha.json` e `ficha.md` (se puder produzir), `fontes.json` (URLs, títulos, verificação/licenças), `manifesto.json` (chave, arquivos, SHA-256, resultados e status) e `pesquisa-ficha.md` quando necessário. Nome: `<CHAVE>-<slug>-<YYYYMMDD-HHMMUTC>.zip`. Guardar em `operational/outputs/geracao-criaturas/<YYYY-MM-DD>/` **fora de docs/** para não publicar conteúdo não homologado no site.
9. Validar ZIP reabrindo-o, conferir integridade, imagens, JSON válido e hashes. Gravar o ZIP em armazenamento persistente acessível ao operador; usar GitHub apenas se a integração suportar upload binário. Falha no upload não é sucesso.
10. Atualizar ambas as filas **somente após** verificação do artefato. Imagem: `PENDENTE=NAO` apenas se a imagem nova existe e está verificada; ficha: `PENDENTE=NAO` somente se a ficha completa foi efetivamente produzida e verificável. Conservar estados parciais de modo idempotente, marcar timestamps, caminhos e erro caso ocorram. Confirmar nova revisão remota dos CSVs, não usar estado em memória obsoleto.
11. Relatar job-ID, CHAVE, espécie, fontes, status individual de imagem e ficha, path/link do ZIP, commit(s), itens faltantes, verificações realizadas. **Nunca alegar criação, publicação ou armazenamento sem confirmação.**

## Execução e limites
- Esperado: até quatro espécies diferentes nesta janela, em ordem ascendente das imagens pendentes.
- Uma execução pode levar mais de 15 minutos; reservas persistentes devem impedir sobreposição.
- **Automação e produção são diferentes**: não é garantido que uma tarefa agendada do ChatGPT tenha gerador de imagens, navegador, acesso de escrita ou upload de ZIP. Valide capacidades antes de ativar os jobs. Não declare que os jobs já estão criados por haver este prompt.
- Não alterar fichas SRD originais nem homologar materiais externos automaticamente. Não trocar uma criatura por outra nem inventar licenças ou estatísticas.
- Ao terminar, apresentar resumo dos quatro jobs com resultado `SUCESSO`, `PARCIAL`, `BLOQUEADO` ou `FALHA`.

## Variante curta para cada execução
"Execute um ciclo do procedimento em `docs/catalogo/relatorios/FLUXO_4_TAREFAS_20261016.md`, reservando atomicamente a próxima imagem pendente dos CSVs `docs/catalogo/relatorios/fila-imagens-1320-20261016.csv` e `docs/catalogo/relatorios/fila-fichas-1320-20261016.csv`. Gere a imagem 2D premium, pesquise a ficha da mesma criatura, faça ZIP com fontes e checksums, persista-o, atualize cada `PENDENTE` conforme evidência e reporte o link do arquivo. Job: `<JOB-ID>`."
