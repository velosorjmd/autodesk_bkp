Documento: TECHNICAL_OVERVIEW.md
Projeto: Automatic Desktop Backup Application (ADBA)
Versao: 0.1 (draft)
Ultima atualizacao: 2025-12-26

1. Proposito

Este documento consolida a visao tecnica do ADBA com foco em arquitetura, componentes e fluxos
principais. Ele deriva do escopo e objetivos definidos em `project_overview.md` e detalha o
conteudo tecnico descrito em `README.md`. Quando houver conflitos, o Project Overview e o README
tem prioridade (conforme `AGENTS.md`).

2. Escopo Tecnico

2.1 In scope (MVP)
- Aplicativo desktop (UI) para configurar jobs, visualizar status/historico e iniciar restauracao.
- Daemon/servico em background para agendamento e execucao de backups.
- Destinos iniciais: local, USB, NAS e pelo menos um destino de nuvem tipo object storage.
- Criptografia de dados em repouso e integracao com keystore do sistema operacional.
- Persistencia local de metadados (ex.: SQLite).
- Observabilidade basica (logs estruturados e metricas minimas).

2.2 Out of scope (fase inicial)
- Backup de imagem completa de disco (bare-metal).
- Console web multi-tenant ou gestao centralizada.
- Apps mobile nativos.
- Sincronizacao em tempo real estilo "drive".

3. Arquitetura de Alto Nivel

3.1 Camadas
- Presentation Layer (UI): configura jobs, mostra status e conduz restauracoes.
- Application Layer (Core Orchestration): coordena casos de uso e regras de negocio.
- Domain Layer (Backup Engine e modelos): logica de backup/restauracao e regras de dominio.
- Infrastructure Layer: conectores de storage e persistencia local.
- Security Layer: criptografia e integracao com keystore do SO.
- System Integration Layer: integracao com servicos/daemons e APIs do SO.

3.2 Principais componentes
- UI Desktop
- Core/Orquestrador
- Backup Engine
- Storage Connectors (local, USB, NAS, nuvem)
- Security & Crypto Layer
- Persistence & Metadata (SQLite ou equivalente)
- System Integration (service/daemon, permissao, notificacoes)

4. Modelo de Dados (conceitual)

Entidades principais (resumo tecnico):
- Job: configuracao de fontes, destinos, politica de retencao e agendamento.
- BackupRun: execucao de um job com timestamps e resultado.
- BackupVersion: snapshot logico de arquivos em um ponto no tempo.
- FileEntry: caminho logico, hash, tamanho e metadados.
- Blob: unidade armazenada no destino, com hash e flags de criptografia.
- StorageTarget: destino e configuracoes especificas (path, endpoint, credenciais).

TODO: validar atributos finais de cada entidade com o time de engenharia.

5. Fluxos Principais

5.1 Criacao/edicao de job
- UI coleta fontes, destinos e politica de retencao.
- Core valida campos obrigatorios e permissao basica.
- Persistencia grava configuracao.
- System Integration registra agendamento.

5.2 Execucao de backup (agendado ou on-demand)
- Daemon dispara o job conforme agendamento.
- Backup Engine descobre arquivos alterados.
- Para cada arquivo: leitura, criptografia (e compressao se configurada), geracao de blobs.
- Storage Connectors escrevem blobs no destino.
- Persistencia registra BackupRun, BackupVersion e estatisticas.
- Logs/metricas são emitidos.

5.3 Restauracao
- UI seleciona job/versao e itens a restaurar.
- Core dispara fluxo de restore.
- Backup Engine recupera blobs, decripta e reconstroi arquivos.
- Logs/metricas de restore são registrados.

5.4 Retencao/garbage collection
- Core executa rotina periodica conforme politica.
- Remove versoes antigas e blobs nao referenciados.

6. Seguranca e Criptografia

- Dados de backup devem estar criptografados em repouso.
- Chaves e segredos devem ser armazenados no keystore do SO.
- Integridade baseada em hash de blob ou de chunk.
- Logs devem evitar dados sensiveis.

TODO: definir algoritmos e modos de criptografia suportados.
TODO: definir politica de rotacao de chaves.

7. Storage Connectors

Conectores iniciais:
- Local
- USB
- NAS
- Nuvem (object storage, provedor a definir)

Requisitos gerais:
- Timeouts e retry.
- Validacao basica de disponibilidade.
- Integracao com Security Layer para criptografia/decriptografia.

TODO: definir contratos de interface e codigos de erro.

8. Observabilidade

8.1 Logs
- Eventos criticos: inicio/fim de backup, falhas por destino, falhas de criptografia/keystore,
  restauracoes.

8.2 Metricas
- Por job: sucesso/falha, volume de dados, duracao.
- Globais: numero de jobs ativos, destinos com falha recorrente.

9. Testes e Qualidade

9.1 Niveis de teste
- Unitarios para componentes de dominio.
- Integracao para backup/restore com destinos diferentes.
- E2E para UI + daemon.

9.2 Casos criticos (minimo)
- Backup incremental apos alteracao pequena.
- Falha de rede durante upload para nuvem e recuperacao.
- Restauracao parcial de um job grande.
- Restauracao para path alternativo sem sobrescrever indevidamente.

10. Decisoes em Aberto / TODOs

- Escolha da stack de UI e linguagem principal.
- Politica de chunking/deduplicacao no MVP.
- Estratégia de observabilidade integrada a ferramentas corporativas.
- Definicao final de algoritmos de criptografia.

11. Referencias

- `project_overview.md`
- `README.md`
- `AGENTS.md`
