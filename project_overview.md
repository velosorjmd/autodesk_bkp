> \*\*Documento:\*\* PROJECT\_OVERVIEW.md

> \*\*Projeto:\*\* Automatic Desktop Backup Application (ADBA)

> \*\*Propósito:\*\* Descrever o contexto, objetivos, escopo, riscos e direcionadores de alto nível do projeto, para alinhamento entre áreas de negócio, TI (Tecnologia da Informação) e engenharia.

> \*\*Uso:\*\* Documento interno, não é material de marketing.



\## 1. Informações Gerais



\* \*\*Nome do projeto:\*\* Automatic Desktop Backup Application (ADBA)

\* \*\*Tipo de projeto:\*\* Other – Aplicativo desktop + daemon/serviço em background (ferramenta interna)

\* \*\*Área de negócio / cliente interno ou externo:\*\* A definir (ex.: TI Corporativa / Segurança da Informação / Equipes de Engenharia)

\* \*\*Sponsor:\*\* A definir (nome + cargo do patrocinador executivo)

\* \*\*Technical Owner / Project Owner:\*\* A definir (ex.: Tech Lead / Engenheiro responsável / Product Owner)

\* \*\*Data / versão do documento:\*\* 26/12/2025 – Versão 1.4



\### 1.1 Uso e manutenção deste documento



\* Descreve \*\*visão, contexto e decisões de alto nível\*\*.

\* Detalhes de requisitos e arquitetura estão em `TECHNICAL\_OVERVIEW.md` e `README.md`.

\* Em caso de conflito:



&nbsp; \* \*\*Intenção de produto e escopo geral:\*\* este Project Overview prevalece.

&nbsp; \* \*\*Detalhes de implementação:\*\* prevalecem Technical Overview / README.

\* Recomenda-se atualizar:



&nbsp; \* No início de cada grande fase (ex.: antes de planejar o MVP, antes de grandes expansões).

&nbsp; \* Sempre que objetivos de negócio, público-alvo ou contexto organizacional mudarem.



\## 2. Contexto e Problema / Oportunidade



Ambientes de trabalho costumam depender de soluções de backup \*\*manuais e fragmentadas\*\*: cópias em discos externos, scripts caseiros, recursos nativos de cada sistema operacional ou ausência total de backup. Isso gera \*\*risco elevado de perda de dados\*\* (falha de hardware, ransomware, erro humano, remoção acidental) e dificulta a recuperação rápida.

Em organizações com múltiplas plataformas (Windows, macOS e futuramente Linux), a situação é agravada por ferramentas heterogêneas, políticas pouco padronizadas de retenção, criptografia e monitoramento, além de baixa visibilidade consolidada para auditoria, governança e conformidade.

O ADBA surge como oportunidade de oferecer um \*\*aplicativo único, multiplataforma, com backup automático e restauração guiada\*\*, com boas práticas de segurança (criptografia forte, uso de keystore do sistema operacional para segredos) e arquitetura extensível (novos destinos e políticas plugáveis). O foco principal são \*\*estações de trabalho em contexto profissional\*\*, incluindo máquinas de desenvolvedores e colaboradores que lidam com arquivos sensíveis ou de alto valor.



\## 3. Definições-Chave



\* \*\*Backup job:\*\* configuração que define o que será copiado (pastas/arquivos de origem), quando (agendamento/política) e para onde (destino).

\* \*\*Retention policy (política de retenção):\*\* regra que define por quanto tempo versões de backup são mantidas e quando podem ser descartadas.

\* \*\*Daemon/service:\*\* processo em segundo plano que executa tarefas de forma contínua ou agendada, sem interação direta do usuário, como o processo que realiza backups em background.

\* \*\*Storage connector (conector de armazenamento):\*\* componente que encapsula como o ADBA lê e escreve dados em um tipo específico de destino (disco local, USB, NAS, nuvem).

\* \*\*End user (usuário final):\*\* pessoa que utiliza o aplicativo desktop para configurar backups e restaurar arquivos na própria estação de trabalho.



\## 4. Objetivos do Projeto



\### 4.1 Objetivo principal



Garantir \*\*backup automatizado e seguro\*\* de arquivos em estações de trabalho (Windows e macOS, com arquitetura preparada para Linux), reduzindo risco de perda de dados e facilitando restauração rápida em falhas ou incidentes.



\### 4.2 Objetivos secundários



\* Oferecer \*\*experiência simples de configuração\*\* de jobs de backup para usuários finais e equipes internas (TI, engenharia).

\* Padronizar \*\*políticas de retenção e versionamento\*\* de arquivos, com suporte a múltiplos destinos (local, USB, NAS, nuvem).

\* Usar mecanismos de segurança do sistema operacional para \*\*armazenar segredos e chaves\*\*, evitando reinventar criptografia e gestão de credenciais.

\* Permitir \*\*extensibilidade\*\* via novos conectores de armazenamento e políticas de backup, sem reescrever o núcleo.

\* Habilitar \*\*monitoramento básico\*\* de backups (status, falhas, volume de dados) para decisões operacionais.



\### 4.3 Métricas de sucesso (KPIs – Key Performance Indicators)



Propostas para validação com negócio e TI:



\* \*\*Cobertura de backup recente:\*\* % de máquinas com ao menos um backup bem-sucedido nas últimas 24/48h, medido por logs do daemon e relatórios agregados.

\* \*\*Confiabilidade de execução:\*\* % de jobs concluídos sem erro em janela definida (ex.: últimas duas semanas), medido por registros de execução.

\* \*\*Tempo de restauração (MTTR – Mean Time to Restore):\*\* tempo médio para restaurar um arquivo em cenários típicos (simulações e incidentes).

\* \*\*Uso correto do produto:\*\* % de usuários com pelo menos \*\*um job de backup válido e ativo\*\* após X dias de instalação, baseado em configuração local e telemetria permitida.

&nbsp; Esses KPIs avaliam se o ADBA \*\*reduz risco e facilita recuperação\*\*, não apenas se o software está tecnicamente funcionando.



\## 5. Escopo do Projeto



\### 5.1 In Scope



Itens que o projeto \*\*entregará na fase inicial / MVP (Minimum Viable Product – Produto Mínimo Viável)\*\*:



\* \*\*Aplicativo desktop multiplataforma\*\* (UI – User Interface, interface com o usuário) para:



&nbsp; \* Configurar jobs de backup (pastas, destinos, políticas).

&nbsp; \* Visualizar status, histórico e versões disponíveis para restauração.

&nbsp; \* Iniciar fluxos de restauração guiada.

\* \*\*Daemon/serviço em background\*\* para:



&nbsp; \* Monitorar arquivos/pastas de origem configuradas.

&nbsp; \* Orquestrar execução de backups conforme políticas.

&nbsp; \* Registrar logs e métricas de execução.

\* Suporte a \*\*destinos iniciais de armazenamento\*\*:



&nbsp; \* Disco local (outro volume interno).

&nbsp; \* Discos externos/USB.

&nbsp; \* NAS (Network-Attached Storage).

&nbsp; \* Pelo menos um provedor de armazenamento em nuvem tipo “object storage” (sem detalhar fornecedor).

\* Camada de \*\*Security \& Crypto\*\*:



&nbsp; \* Criptografia de dados em repouso com algoritmos fortes.

&nbsp; \* Gestão de chaves e segredos integrada a mecanismos do sistema operacional (ex.: Windows Credential Manager, macOS Keychain).

\* Camada de \*\*Persistence \& Metadata\*\*:



&nbsp; \* Banco local (ex.: SQLite) para metadados de versões, localização de blobs de backup e estado de jobs.

\* \*\*Observabilidade básica\*\*:



&nbsp; \* Logs estruturados de backup/restauração.

&nbsp; \* Métricas mínimas (sucesso/falha, duração, volume de dados).



\### 5.2 Out of Scope



Itens \*\*deliberadamente excluídos\*\* desta fase:



\* Aplicativos mobile nativos (iOS/Android).

\* Gestão centralizada multi-tenant (console web com visão global de máquinas) – potencial evolução futura.

\* Backup de \*\*imagem completa de disco\*\*/bare-metal; foco em arquivos e diretórios de dados de usuário.

\* Sincronização em tempo real (file sync tipo “drive em nuvem”); foco em backup versionado e agendado.

\* Recursos de \*\*comercialização externa\*\* (licenciamento, billing, portal público).

\* Funcionalidades de \*\*Data Loss Prevention (DLP)\*\* e \*\*antivírus/EDR (Endpoint Detection and Response)\*\*; o ADBA não substitui essas soluções.



\## 6. Público-Alvo e Stakeholders



\### 6.1 Usuários finais / público-alvo



\* Usuários de estações de trabalho \*\*Windows 10+ (64-bit)\*\* e \*\*macOS 12+\*\* com arquivos de trabalho críticos (código, documentos, artefatos).

\* Equipes técnicas (engenharia, QA – Quality Assurance, data/infra) que desejam \*\*padronizar backup de ambientes de desenvolvimento\*\*.

\* Colaboradores com arquivos sensíveis ou de alto valor (relatórios financeiros, documentos jurídicos, artefatos de projeto).

&nbsp; O contexto primário assumido é \*\*uso profissional/corporativo em estações “pessoal, porém gerenciada”\*\*, ainda que seja possível uso individual por desenvolvedores ou times pequenos.



\### 6.2 Principais stakeholders e papéis



\* \*\*TI / Infraestrutura:\*\* define diretrizes de backup em estações, apoia implantação, distribuição e suporte de primeiro nível.

\* \*\*Segurança da Informação:\*\* define requisitos mínimos de criptografia, chaves e retenção compatíveis com regras internas; valida riscos e controles compensatórios.

\* \*\*Equipes de Engenharia / Desenvolvimento:\*\* usuários técnicos e influenciadores de requisitos de performance, developer experience e extensibilidade; fornecem feedback contínuo.

\* \*\*Suporte / Service Desk:\*\* auxilia instalação, troubleshooting básico e recuperação em incidentes.

\* \*\*Gestão de Produto / Tech Lead:\*\* prioriza roadmap, garante alinhamento com objetivos de negócio e toma decisões de trade-off relevantes.



\## 7. Visão Geral da Solução



O ADBA é composto por um \*\*aplicativo desktop\*\* (UI) para configurar jobs de backup, visualizar status e restaurar arquivos, e por um \*\*daemon/serviço em background\*\* responsável pela execução contínua e resiliente dos backups conforme as políticas definidas.



\### 7.1 Princípios de design



\* \*\*Security by default:\*\* criptografia ativa para dados de backup e uso de mecanismos nativos do sistema operacional sempre que possível.

\* \*\*Transparência para o usuário final:\*\* status claros, alertas em caso de falha e indicação explícita de ausência de backups recentes.

\* \*\*Extensibilidade via conectores:\*\* novos destinos de armazenamento e políticas de backup adicionados principalmente por novos “connectors” e configurações, evitando reescritas de núcleo.

\* \*\*Mínimo atrito na rotina:\*\* agendamento e execução de backups para minimizar impacto em performance e produtividade.



\### 7.2 Principais componentes (alto nível)



\* \*\*UI Layer:\*\* interface desktop multiplataforma para configuração, status/histórico e restauração.

\* \*\*Application Layer (Core Orchestration):\*\* coordena fluxo de backup/restauração, aplica regras de negócio e orquestra os componentes.

\* \*\*Backup Engine:\*\* detecta mudanças, monta conjuntos de arquivos, gera versões e valida integridade.

\* \*\*Storage Connectors:\*\* interagem com destinos (local, USB, NAS, nuvem), com timeout, retry e verificações básicas.

\* \*\*Security \& Crypto Layer:\*\* criptografa/descriptografa dados e integra mecanismos de armazenamento seguro de credenciais do sistema operacional.

\* \*\*Persistence \& Metadata Layer:\*\* mantém índices de versões, manifests, histórico e estado de jobs em banco local (ex.: SQLite).

\* \*\*System Integration Layer:\*\* integra com o sistema operacional (registro de serviço/daemon, agendamento, permissões e notificações).



\## 8. Premissas e Dependências



\### 8.1 Premissas



\* Usuários terão \*\*espaço de armazenamento suficiente\*\* nos destinos escolhidos para suportar a política de retenção.

\* A rede (corporativa ou doméstica) terá \*\*capacidade razoável\*\* para transferir volumes de dados dentro das janelas de backup.

\* Sistemas operacionais suportados (Windows 10+, macOS 12+) estarão \*\*atualizados\*\* para fornecer APIs de segurança e serviços necessários.

\* Stakeholders de negócio e técnicos estarão disponíveis para \*\*validações periódicas\*\* de escopo, priorização e design.



\### 8.2 Dependências



\* \*\*Infraestrutura de NAS e nuvem\*\* provida por TI ou fornecedor externo, com disponibilidade e capacidade adequadas.

\* \*\*Políticas corporativas de segurança e privacidade\*\* (classificação de dados, requisitos de criptografia, retenção mínima/máxima) definidas por Segurança da Informação e Compliance.

\* Acesso às APIs de \*\*keystore do sistema operacional\*\* (Credential Manager, Keychain, etc.) em cada plataforma suportada.

\* Suporte de \*\*equipes de distribuição/instalação\*\* (ex.: ferramentas de gestão de dispositivos) para rollout em escala em ambientes corporativos.



\## 9. Riscos Iniciais e Mitigações



\* \*\*Risco 1 – Criptografia e gestão de chaves incorretas\*\*



&nbsp; \* Impacto: High | Probabilidade: Medium

&nbsp; \* Mitigação: revisões de código em Security \& Crypto Layer, testes de backup/restore com rotação de chaves, uso consistente de mecanismos nativos do sistema operacional.

\* \*\*Risco 2 – Degradação de performance nas máquinas\*\*



&nbsp; \* Impacto: Medium | Probabilidade: Medium

&nbsp; \* Mitigação: implementação incremental/assíncrona, agendamento em horários de menor uso, opções de throttling configuráveis, testes de carga realistas.

\* \*\*Risco 3 – Complexidade multiplataforma (Windows, macOS, Linux)\*\*



&nbsp; \* Impacto: Medium | Probabilidade: Medium

&nbsp; \* Mitigação: abstrações claras de integração com SO, priorização de suporte pleno a Windows/macOS na primeira fase, suíte de testes automatizados para caminhos críticos.

\* \*\*Risco 4 – Uso incorreto e falsa sensação de segurança\*\*



&nbsp; \* Impacto: High | Probabilidade: Medium

&nbsp; \* Mitigação: UI com validações e alertas claros, notificações em falhas repetidas, visualizações que destaquem ausência de backups recentes, mensagens educativas no produto.

\* \*\*Risco 5 – Baixa adoção e configuração incompleta\*\*



&nbsp; \* Impacto: Medium | Probabilidade: Medium

&nbsp; \* Mitigação: onboarding guiado, sugestões de jobs iniciais, documentação de “primeiros passos” e acompanhamento via KPIs de adoção.

\* \*\*Risco 6 – Tratamento inadequado de dados sensíveis\*\*



&nbsp; \* Impacto: High | Probabilidade: Low–Medium

&nbsp; \* Mitigação: alinhamento com políticas internas de privacidade e classificação de dados, possibilidade de excluir certos diretórios (ex.: pastas proibidas), comunicação clara sobre o que é apropriado incluir em backups.



\## 10. Timeline e Marcos (High-Level)



> Datas são placeholders e devem ser ajustadas ao planejamento real.



\* \*\*Planned start:\*\* \[dd/mm/aaaa]

\* \*\*Planned end:\*\* \[dd/mm/aaaa]



\*\*Marcos propostos:\*\*



\* \*\*Milestone 1 – Discovery e escopo detalhado definidos\*\*



&nbsp; \* Critérios: objetivos de negócio validados, escopo In/Out-of-Scope acordado, principais riscos mapeados.

\* \*\*Milestone 2 – MVP com destinos locais/USB em ambiente de testes\*\*



&nbsp; \* Critérios: backup/restauração básicos funcionando, métricas iniciais de sucesso/falha, documentação mínima de instalação.

\* \*\*Milestone 3 – Suporte a NAS + primeiro conector de nuvem e hardening de segurança\*\*



&nbsp; \* Critérios: conectores adicionais funcionando, revisão de segurança realizada, testes de carga básicos completos.

\* \*\*Milestone 4 – Release inicial para grupo piloto / produção controlada\*\*



&nbsp; \* Critérios: implantação em grupo limitado, KPIs básicos monitorados, feedback coletado para roadmap.



\## 11. Entregáveis Principais



\* Aplicativo desktop (UI) + daemon/serviço para Windows e macOS com:



&nbsp; \* Configuração de jobs de backup.

&nbsp; \* Painel de status/histórico.

&nbsp; \* Fluxos de restauração guiada.

\* Conjunto inicial de \*\*Storage Connectors\*\*:



&nbsp; \* Local/USB.

&nbsp; \* NAS.

&nbsp; \* Pelo menos um conector de armazenamento em nuvem.

\* Camada de \*\*Security \& Crypto\*\* integrada ao keystore do sistema operacional, com documentação de uso e parâmetros recomendados.

\* Artefatos de \*\*documentação\*\*:



&nbsp; \* Guia de instalação e operação (TI / suporte).

&nbsp; \* Guia rápido para usuários finais (configuração e validação de backup).

&nbsp; \* Documentação técnica de arquitetura e componentes (README / Technical Overview).

\* Conjunto mínimo de \*\*testes automatizados\*\* (unitários, integração, smoke) e instruções para execução em CI (Continuous Integration – Integração Contínua).



\## 12. Notas Finais / Decisões-Chave



\* Arquitetura em camadas, com UI desacoplada de conectores e lógica de criptografia, para reduzir acoplamento e riscos de segurança.

\* \*\*Segredos, tokens e chaves\*\* não serão armazenados junto aos dados de backup; sempre que possível, serão mantidos em mecanismos seguros do sistema operacional.

\* Versão inicial focada em \*\*destinos locais/USB/NAS + pelo menos um destino de nuvem\*\*, com suporte a Linux considerado no design, mas não exigido no primeiro release.

\* Este Project Overview é a referência para decisões de escopo, objetivos e trade-offs de alto nível; detalhes de implementação residem em documentos técnicos específicos.



