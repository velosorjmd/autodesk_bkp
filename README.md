1. Metadados

Versão: 1.1

Status: Draft técnico (pronto para implementação incremental)

Última atualização: 2025-12-26

Escopo: Arquitetura, decisões de design, componentes, fluxos e validação (engenharia, QA, DevOps, segurança).

2. Propósito e Contexto
2.1 Propósito deste documento

Este documento descreve a solução técnica do ADBA (Automatic Desktop Backup Application) em nível de engenharia: arquitetura, componentes principais, decisões estruturais, riscos técnicos e critérios de validação.
Ele é voltado para desenvolvedores, engenheiros de infraestrutura, QA (Quality Assurance), DevOps e segurança.

2.2 Problema a ser resolvido

Estações de trabalho (principalmente Windows e macOS) frequentemente não contam com um mecanismo de backup de arquivos de trabalho que seja:

Confiável.

Automatizado.

Criptografado.

Fácil de restaurar.

Hoje, usuários costumam depender de:

Cópias manuais em discos externos.

Scripts ad-hoc.

Ferramentas nativas heterogêneas por sistema operacional.

Ausência total de backup estruturado.

Isso gera riscos de:

Perda de dados (falha de disco, ransomware, erro humano).

Recuperação lenta ou impossível.

Descumprimento de políticas internas de segurança.

O ADBA busca oferecer um mecanismo consistente de backup/restauração de arquivos de usuário, com arquitetura preparada para múltiplas plataformas e destinos (local, USB, NAS, nuvem).

3. Visão Geral do Sistema
3.1 Objetivo técnico

Fornecer um aplicativo desktop e um daemon/serviço em background capazes de:

Configurar “jobs de backup” (origens, destinos, políticas).

Executar backups automáticos e versionados.

Restaurar arquivos com uma experiência guiada.

Manter metadados consistentes sobre o que foi armazenado, onde e em qual versão.

Proteger dados e credenciais com criptografia forte e keystores do sistema operacional.

3.2 Definições técnicas chave

Job de backup: configuração que agrupa fontes (pastas/arquivos), destino(s), política de retenção e agendamento.

Versão de backup: conjunto de blobs (arquivos ou chunks) + metadados que representam o estado de um conjunto de arquivos em um momento no tempo.

Storage Connector: componente responsável por falar com um destino específico (ex.: disco local, USB, NAS, nuvem S3-like).

Daemon/serviço: processo de longa duração que executa backups conforme agendamento, sem interação ativa do usuário.

3.3 Componentes principais (high level)

UI Desktop: interface para criação/edição de jobs, visualização de status, histórico e restauração.

Core / Orquestrador: coordena jobs, aplica regras de negócio (política de retenção, estados, validações) e conversa com os demais componentes.

Backup Engine: responsável pelo pipeline de backup: descoberta de arquivos, leitura, compressão/opcional, criptografia, chunking (se existir) e envio via Storage Connectors.

Storage Connectors: abstrações de IO para destinos (local, USB, NAS, nuvem).

Security & Crypto Layer: abstração de criptografia/assinatura, gestão de chaves e integração com o keystore/secret store do sistema operacional.

Persistence & Metadata: banco local (ex.: SQLite) para status de jobs, manifests de versões, índices de blobs/arquivos, logs técnicos.

System Integration: integração com SO (registro de serviço, permissões, caminhos padrão, notificações).

4. Arquitetura Lógica
4.1 Camadas principais

Presentation Layer (UI):

Aplicativo desktop que roda na máquina do usuário.

Permite criar/editar jobs, visualizar health de backup e iniciar restaurações.

Application Layer (Core):

Expõe casos de uso principais: CreateJob, UpdateJob, RunJobNow, PauseJob, Restore.

Mantém consistência entre configurações, agendamentos e registros de execução.

Domain Layer (Backup Engine + modelos):

Define entidades: Job, BackupRun, BackupVersion, FileEntry, Blob, StorageTarget.

Implementa regras de negócio de backup/restauração, chunking (se usado), deduplicação (se existir), retenção e integridade.

Infrastructure Layer:

Storage Connectors (local, USB, NAS, nuvem).

Repositórios persistentes (SQLite ou equivalente).

Adaptações para logging, métricas, telemetria.

Security Layer:

Criptografia de payloads.

Gestão de chaves e segredos (por sistema operacional).

Assinatura/verificação opcional para integridade.

System Integration Layer:

Registro e controle do daemon/serviço (Windows Service, launchd, etc.).

Interação com APIs de permissões de arquivos, notificações e event logs.

4.2 Fluxo de responsabilidade entre camadas

UI chama casos de uso expostos pelo Core.

Core consulta/atualiza dados via Persistence & Metadata.

Core delega a execução de backup/restauração ao Backup Engine.

Backup Engine interage com System Integration (prioridade de IO, limites) e com Storage Connectors.

Storage Connectors usam Security & Crypto para criptografar/decriptar dados antes de escrever/ler.

5. Modelagem de Dados e Metadados
5.1 Entidades principais (conceitual)

Job:

ID, nome, descrição.

Lista de origens (paths).

Lista de destinos (StorageTargets).

Política de retenção (por tempo, por quantidade de versões ou ambos).

Política de agendamento (janela, frequência).

Flags de performance (prioridade, limites de banda, parallelism).

BackupRun:

ID, JobId.

Timestamp de início/fim.

Resultado (Success, PartialSuccess, Failure).

Estatísticas (arquivos processados, bytes lidos, bytes escritos, tempo total).

BackupVersion:

ID, JobId.

Identificador de “snapshot lógico” (ex.: data/hora, geração).

Relacionamento com FileEntries e Blobs associados.

FileEntry:

Caminho lógico no sistema de arquivos.

Hash do conteúdo e/ou hash por chunk.

Tamanho.

Metadados adicionais (permissões, timestamps relevantes).

Blob:

ID, localização física no StorageTarget.

Hash (para integridade e deduplicação).

Tamanho.

Flags de criptografia (algoritmo, versão de chave).

StorageTarget:

Tipo (Local, USB, NAS, Cloud).

Configurações específicas (path base, endpoint, credenciais).

Estado de health (último teste de conexão, falhas recentes).

5.2 Banco de dados local

Provável escolha: SQLite, devido a:

Baixa complexidade operacional.

Suporte multiplataforma.

Bom suporte em linguagens comuns de desktop.

Uso principal: metadados de jobs, versões, blobs, histórico de execuções, configuração de usuário e cache de health dos destinos.

6. Fluxos Principais
6.1 Criação e edição de Job

Usuário abre UI e cria um novo job de backup.

Define origens (pastas/arquivos), destino(s) e política de retenção.

Escolhe agendamento (ex.: diariamente às 02:00).

UI valida campos obrigatórios e permissões básicas (ex.: path acessível).

Core persiste Job e configura agendamento.

Opcionalmente, é oferecido um “teste rápido” (dry run ou backup inicial) para validar.

6.2 Execução de backup (agendado ou on-demand)

Daemon acorda o job conforme cronograma ou por comando explícito (RunJobNow).

Backup Engine coleta lista de arquivos a processar (comparação incremental, hashes, timestamps).

Para cada arquivo:

Lê conteúdo.

Aplica criptografia/compressão conforme política.

Gera blobs (inteiros ou chunked).

Envia para destino via Storage Connector.

Atualiza metadados (BackupVersion, FileEntries, Blobs).

Gera eventos (BackupStarted, BackupCompleted, BackupFailed).

Atualiza estatísticas, logs e métricas.

6.3 Restauração

Usuário abre UI em modo restore.

Escolhe Job e Version a restaurar.

Lista de arquivos é exibida com opção de filtragem (por path, nome, data).

Usuário seleciona arquivos/pastas alvo.

Core solicita ao Backup Engine o fluxo de restore.

Backup Engine obtém blobs via Storage Connectors, decripta e reescreve arquivos em destino escolhido (path original ou alternativo).

Logs e eventos de restauração são registrados (RestoreCompleted ou falhas).

6.4 Gestão de retenção

Periodicamente, o Core executa rotinas de “garbage collection” de versões, removendo versões antigas conforme a política de retenção.

Ao remover uma versão:

Verifica-se se blobs são compartilhados por outras versões (deduplicação).

Apenas blobs não referenciados por mais nenhuma versão são efetivamente removidos.

7. Segurança e Conformidade
7.1 Objetivos de segurança

Garantir confidencialidade dos dados de backup.

Minimizar risco de exposição de segredos (credenciais de storage, chaves).

Garantir integridade dos dados (detectar corrupção ou adulteração).

Aderir a requisitos mínimos de políticas internas (criptografia forte, retenção, descarte seguro).

7.2 Criptografia

Dados de backup devem ser criptografados em repouso, idealmente com algoritmos simétricos fortes (ex.: AES-256 em modo autenticado).

Chaves não são armazenadas em texto puro em disco, e o produto não implementa keystore próprio quando o sistema operacional oferece um mecanismo nativo seguro.

7.3 Gestão de credenciais

Integração com keystores nativos (ex.: Windows Credential Manager, macOS Keychain) para armazenar:

Tokens de acesso à nuvem.

Senhas de NAS (quando aplicável).

Chaves simétricas (ou segredos para derivação de chave).

Em cenários sem keystore nativo adequado, utilizar mecanismo mínimo de criptografia local com senha do usuário e alertar explicitamente sobre o risco aumentado.

7.4 Integridade e auditoria

Todos os blobs possuem hash (conteúdo ou por chunk).

Logs de operações relevantes:

Criação/edição de jobs.

Execução de backups.

Restaurações.

Falhas de acesso a destino.

Logs devem evitar registrar dados sensíveis em claro (ex.: paths sensíveis, credenciais).

8. Desempenho, Escalabilidade e Limitações
8.1 Desempenho esperado

Projeto focado em estações individuais, não em data centers.

Expectativa:

Não saturar IO de disco nem rede de forma permanente.

Permitir agendamento em janelas de menor uso (ex.: madrugada).

Possibilidade de throttling configurável por job (limites de banda e paralelismo).

8.2 Estratégias de otimização (presentes ou planejadas)

Leitura e escrita em pipeline (bufferizado, assíncrono).

Processamento incremental (somente arquivos alterados).

Deduplicação opcional por hash de arquivo ou chunk.

Paralelismo controlado por destino (ex.: limitar número de conexões simultâneas a um bucket de nuvem).

8.3 Limitações conhecidas

Foco inicial em backup de arquivos, não de imagens completas de disco.

Foco em estações individuais; gestão centralizada não faz parte do escopo inicial.

Dependência de infraestrutura externa de storage (NAS, nuvem) fora do controle direto do ADBA.

9. Observabilidade e Telemetria
9.1 Logs

Níveis sugeridos: DEBUG, INFO, WARN, ERROR.

Eventos críticos:

Início/fim de backup.

Falhas por destino (com códigos de erro).

Falhas de criptografia ou de acesso ao keystore.

Restaurações iniciadas/concluídas.

9.2 Métricas

Por job:

Execuções bem-sucedidas vs. falhas.

Volume total de dados transferidos.

Tempo médio de execução.

Métricas globais:

Número de jobs ativos.

Número de destinos com falhas recorrentes.

9.3 Integração com ferramentas externas

Export de logs e métricas para ferramentas de observabilidade pode ser adicionado via adaptadores (ex.: integração futura com agentes de log/metrics já usados na organização).

10. Testes e Qualidade
10.1 Níveis de teste

Unitários: focados em componentes de domínio (Backup Engine, políticas de retenção, criptografia).

Integração:

Backup/restore fim a fim em ambiente controlado.

Testes com diferentes tipos de destino (local, USB, NAS, nuvem).

End-to-end (E2E):

Cenários completos envolvendo UI + daemon.

Casos de falha controlada (destino offline, falta de espaço, corrupção de arquivo).

10.2 Casos de teste críticos

Backup incremental após pequena alteração em um arquivo grande.

Falha de rede durante upload para nuvem com recuperação posterior.

Restauração parcial (apenas um subdiretório) de um job grande.

Testes de performance sob volume médio e alto (definidos pelo time).

Restauração em path alternativo, garantindo que arquivos não sobrescrevam dados inesperados.

11. Roadmap Técnico (High Level)
11.1 MVP

UI simples para criar/editar jobs.

Daemon com agendador básico.

Destinos: disco local + USB.

Criptografia local simples integrada a keystore (quando disponível).

Banco local com SQLite.

Logs básicos e resultado de execução por job.

11.2 Evoluções planejadas

Suporte a NAS e primeiro conector de nuvem.

Primitivas de deduplicação mais avançadas (por chunk).

Painel de “saúde do backup” com métricas agregadas.

Melhorias de UX (experiência do usuário) na configuração de jobs e restauração.

11.3 Itens em aberto / decisões futuras

Escolha definitiva da linguagem/plataforma UI (ex.: framework desktop específico).

Estratégia padrão de chunking/deduplicação (se será parte do MVP ou não).

Nível de integração com observabilidade corporativa (logs centralizados, métricas).

12. Conclusão

Este documento apresenta a visão técnica de referência do ADBA, traduzindo requisitos em uma arquitetura implementável, com componentes e fluxos claros.
Ele deve evoluir junto com o produto, mantendo coerência com o Project Overview e com o Technical Overview mais detalhado. Todas as mudanças relevantes na arquitetura ou nos componentes principais devem ser refletidas aqui, preservando a rastreabilidade com requisitos funcionais e não funcionais.