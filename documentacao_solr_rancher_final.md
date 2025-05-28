# Documentação Completa: Instalação e Manutenção do Ambiente SOLR no Rancher via GitOps

## Índice

1.  Introdução
2.  Visão Geral do Ambiente Solr no Rancher (via GitOps)
3.  Procedimentos de Instalação e Configuração Principal do Solr (`apps/solr/solr/values.yaml`)
    3.1. Configurações Globais da Aplicação e Imagem do Solr
    3.2. Autenticação do Solr
    3.3. Configurações Java e Otimizações do SOLR
    3.4. Gerenciamento de Réplicas, Agendamento de Pods e Afinidade
    3.5. Alocação de Recursos Computacionais (Limits e Requests)
    3.6. Configuração de Volumes de Dados e Montagens de Arquivos Essenciais
    3.7. Contêineres de Inicialização (Init Containers) para Preparação de Ambiente
    3.8. Configuração de Acesso Externo (Ingress)
    3.9. Persistência de Dados do SOLR (Collections e Índices)
    3.10. Coleta de Métricas com Prometheus para Monitoramento
    3.11. Configuração do Zookeeper para Operação em Modo SolrCloud
    3.12. Política de Rede (NetworkPolicy) para SOLR
4.  Gerenciamento da Aplicação Solr via Fleet (`apps/solr/solr/fleet.yaml`)
5.  Gerenciamento da Política de Segurança Java via Kustomize (`apps/solr/solr/security/kustomization.yaml`)
6.  Análise Detalhada da Política de Segurança Java (`apps/solr/solr/security/security.policy`)
7.  Procedimentos Detalhados de Manutenção do Ambiente Solr
    7.1. Gerenciamento de Backups Automatizados (`apps/solr/solr-backups/`)
        7.1.1. Orquestração dos Backups via Fleet
        7.1.2. Estrutura e Funcionamento Detalhado dos CronJobs de Backup
        7.1.3. Procedimento Prático para Criação ou Modificação de um Job de Backup
    7.2. Gerenciamento Centralizado e Seguro de Credenciais com Vault e External Secrets (`apps/solr/solr-secret/`)
        7.2.1. Orquestração da Configuração de Segredos via Fleet
        7.2.2. Configuração do `SecretStore`: A Ponte para o Vault
        7.2.3. Configuração do `ExternalSecret`: Sincronizando Segredos do Vault para o Kubernetes
        7.2.4. Fluxo de Gerenciamento e Atualização de Segredos
    7.3. Servidor Dedicado para Upload de Plugins e Arquivos Adicionais (`apps/solr/solr-server-upload/`)
        7.3.1. Orquestração do Servidor de Upload via Fleet e Kustomize
        7.3.2. Detalhamento dos Recursos Kubernetes Gerenciados pelo Kustomize
        7.3.3. Configuração do PersistentVolumeClaim (PVC) para a Pasta de Plugins (NFS)
        7.3.4. Análise do Deployment do Servidor de Upload
        7.3.5. Configuração do Ingress para Acesso Externo ao Servidor de Upload
        7.3.6. Configuração do Service Kubernetes para o Servidor de Upload
        7.3.7. Procedimento Detalhado para Upload de Plugins e Considerações de Segurança
    7.4. Construção e Manutenção de Dicionários de Sugestão de Busca (`apps/solr/solr-suggest/`)
        7.4.1. Orquestração dos Jobs de Sugestão via Fleet
        7.4.2. Estrutura e Funcionamento dos CronJobs de Construção de Sugestões
        7.4.3. Análise do Job de Verificação de Status dos Suggesters (`suggest-status.yaml`)
8.  Configuração da Camada de Armazenamento: StorageClass para Solr (`apps/vsphere/sc-solr/`)
    8.1. Orquestração da StorageClass via Fleet
    8.2. Definição e Parâmetros da StorageClass `sc-solr`
9.  Monitoramento, Troubleshooting e Melhores Práticas
    9.1. Monitoramento Efetivo do Ambiente Solr
    9.2. Dicas de Troubleshooting para Problemas Comuns
    9.3. Recomendações e Melhores Práticas
10. Glossário de Termos Técnicos
11. Conclusão

---

## 1. Introdução

Este documento técnico tem como finalidade principal estudar e documentar de forma exaustiva e clara os procedimentos de instalação, configuração e manutenção do ambiente Apache Solr implantado na infraestrutura de contêineres Rancher do Tribunal de Contas da União (TCU). Toda a análise e documentação são estritamente baseadas na configuração existente no repositório GitOps e no ambiente do Rancher.

O Apache Solr é uma plataforma de busca de código aberto, altamente escalável e robusta, construída sobre o Apache Lucene. Sua utilização no TCU é vital para diversas aplicações que requerem capacidades avançadas de indexação e consulta de grandes volumes de dados. A complexidade inerente à sua configuração e manutenção em um ambiente orquestrado como o Kubernetes, gerenciado pelo Rancher, exige uma documentação detalhada e precisa para garantir a sua operacionalidade, segurança e manutenibilidade a longo prazo.

Este manual visa consolidar todas as informações relevantes dispersas em configurações e documentos anteriores, corrigindo inconsistências, preenchendo lacunas e oferecendo um guia coeso e prático para as equipes técnicas. Serão abordados desde os aspectos fundamentais da instalação e configuração do Solr e seus componentes (como Zookeeper), passando pela integração com ferramentas de gerenciamento de segredos (HashiCorp Vault), automação de backups, provisionamento de armazenamento (vSphere), até os procedimentos específicos de manutenção e boas práticas.

O público-alvo deste documento inclui administradores de sistemas, engenheiros de DevOps, desenvolvedores que interagem com o Solr e qualquer profissional técnico do TCU envolvido na gestão, operação ou utilização do ambiente Solr no Rancher. Espera-se que, ao final da leitura, o usuário tenha uma compreensão profunda de como o Solr está configurado, como é gerenciado via GitOps e como realizar os procedimentos necessários para sua manutenção eficaz.

---

## 2. Visão Geral do Ambiente Solr no Rancher (via GitOps)

O ambiente Apache Solr no Tribunal de Contas da União (TCU) é implantado e gerenciado utilizando uma abordagem moderna de GitOps, onde o repositório Git serve como a fonte única da verdade para o estado desejado da aplicação e de toda a sua infraestrutura de suporte. A plataforma Rancher, por meio de seu componente Fleet, desempenha um papel crucial ao monitorar continuamente este repositório Git e aplicar automaticamente quaisquer alterações de configuração aos clusters Kubernetes de destino. Esta metodologia robusta assegura um alto grau de rastreabilidade, versionamento consistente, reprodutibilidade e automação nos processos de implantação, atualização e gerenciamento contínuo do Solr.

A estrutura do repositório GitOps para o Solr é organizada de forma modular, com diretórios específicos dedicados a gerenciar diferentes aspectos e componentes da solução Solr. Essa modularidade facilita a compreensão, o gerenciamento e a manutenção de cada parte do sistema. Os principais diretórios e seus respectivos propósitos, localizados sob `apps/solr/` e `apps/vsphere/`, são:

*   **`apps/solr/solr/`**: Este é o coração da implantação do Solr. Contém a configuração principal da aplicação Solr, incluindo seu ensemble Zookeeper (essencial para o modo SolrCloud). A implantação é realizada utilizando um chart Helm, cujos valores são definidos no arquivo `values.yaml`. O gerenciamento do deployment pelo Fleet é especificado no arquivo `fleet.yaml`. Adicionalmente, um subdiretório `security/` contém a configuração da política de segurança Java do Solr, gerenciada via Kustomize (`kustomization.yaml` e o arquivo `security.policy`).

*   **`apps/solr/solr-backups/`**: Responsável pela automação dos backups das collections (conjuntos de dados indexados) do Solr. Este diretório contém os manifestos YAML para diversos `CronJob` Kubernetes, cada um configurado para realizar o backup de uma collection específica ou de um grupo de collections. Um arquivo `fleet.yaml` agrega e gerencia a implantação desses CronJobs.

*   **`apps/solr/solr-secret/`**: Define a integração segura com o HashiCorp Vault para o gerenciamento centralizado das credenciais sensíveis do Solr, como senhas de administrador. Utiliza o External Secrets Operator do Kubernetes para sincronizar os segredos do Vault com os Segredos nativos do Kubernetes. Os arquivos de configuração incluem `fleet.yaml` para o gerenciamento pelo Fleet, `solr-secret-store.yaml` para definir a conexão com o Vault, e `solr-secret.yaml` para especificar quais segredos devem ser sincronizados.

*   **`apps/solr/solr-server-upload/`**: Configura e implanta um servidor de upload simples, permitindo que plugins customizados e outros arquivos necessários sejam facilmente disponibilizados para os pods do Solr. Este componente é gerenciado por um `fleet.yaml` e um subdiretório `kustomize/` que contém os manifestos Kubernetes para o deployment, serviço, PVC e ingress do servidor de upload.

*   **`apps/solr/solr-suggest/`**: Gerencia a funcionalidade de sugestão de busca (autocompletar) do Solr. Contém manifestos YAML para `CronJob` Kubernetes que reconstroem periodicamente os dicionários de sugestão (suggesters) para diversas collections. Um arquivo `fleet.yaml` orquestra a implantação desses jobs.

*   **`apps/vsphere/sc-solr/`**: Embora fora do diretório `apps/solr/`, este caminho é crucial pois define a `StorageClass` customizada, denominada `sc-solr`. Esta StorageClass é utilizada para o provisionamento dinâmico de volumes persistentes para os dados do Solr e do Zookeeper, e é provavelmente otimizada ou configurada especificamente para a infraestrutura de armazenamento baseada em vSphere do TCU.

Este documento técnico se propõe a explorar cada um desses componentes em profundidade, detalhando suas configurações, interdependências e os procedimentos de manutenção associados, com o objetivo de fornecer um manual completo e prático para as equipes do TCU.

---


## 3. Procedimentos de Instalação e Configuração Principal do Solr (`apps/solr/solr/values.yaml`)

O arquivo `values.yaml` localizado em `apps/solr/solr/values.yaml` é o pilar da configuração da implantação do Solr através do seu chart Helm. Ele define uma vasta gama de parâmetros que controlam desde a imagem do contêiner Solr a ser utilizada, passando por configurações de autenticação, alocação de recursos, persistência de dados, até a configuração do Zookeeper, que é essencial para o modo SolrCloud. A seguir, cada seção principal deste arquivo é detalhada para fornecer uma compreensão completa de como o Solr é configurado no ambiente do TCU.

### 3.1. Configurações Globais da Aplicação e Imagem do Solr

Esta seção do `values.yaml` estabelece parâmetros globais que afetam a implantação e especifica a imagem Docker do Solr a ser utilizada.

-   **Registro de Imagem Global (`global.imageRegistry`):**
    -   **Valor:** `registry.rancher.tcu.gov.br/aplicacoes`
    -   **Descrição Detalhada:** Define o registro de contêiner privado do TCU de onde as imagens Docker, incluindo a do Solr e potencialmente a do Zookeeper, são baixadas. Utilizar um registro privado aumenta a segurança e o controle sobre as imagens em execução.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `global.imageRegistry`)

-   **Permitir Imagens Inseguras (`global.security.allowInsecureImages`):**
    -   **Valor:** `true`
    -   **Descrição Detalhada:** Esta configuração, quando `true`, permite que o Kubernetes baixe imagens de registros que podem não ter um certificado TLS válido (HTTPS). Embora possa facilitar a configuração em ambientes de desenvolvimento ou com registros internos específicos, em produção, é ideal garantir que todos os registros sejam acessados via HTTPS seguro. A implicação aqui é que o registro `registry.rancher.tcu.gov.br` pode estar configurado para ser acessado via HTTP ou com um certificado autoassinado que o Kubernetes, por padrão, não confiaria.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `global.security.allowInsecureImages`)

-   **Nome Completo (Override) (`fullnameOverride`):**
    -   **Valor:** `solr`
    -   **Descrição Detalhada:** Sobrescreve o nome completo gerado pelo Helm para os recursos Kubernetes criados por este chart. Ao definir como `solr`, todos os recursos (Deployments, Services, StatefulSets, etc.) terão nomes previsíveis prefixados ou diretamente nomeados como `solr`, facilitando a identificação e o gerenciamento.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `fullnameOverride`)

-   **Imagem do SOLR (`image`):
    -   **Repositório (`image.repository`):** `solr`
        -   **Descrição Detalhada:** Especifica o nome da imagem Solr a ser buscada no registro definido em `global.imageRegistry`. Portanto, a imagem completa seria `registry.rancher.tcu.gov.br/aplicacoes/solr`.
    -   **Tag (`image.tag`):** `9.7.0-debian-12-r4`
        -   **Descrição Detalhada:** Define a versão específica da imagem Solr a ser utilizada. Usar uma tag explícita em vez de `latest` é uma prática crucial para garantir implantações consistentes e reprodutíveis. Esta tag indica a versão 9.7.0 do Solr, baseada no Debian 12, com uma revisão específica `r4` (provavelmente uma build customizada ou fornecida pelo Bitnami ou outro mantenedor).
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linhas: `image.repository`, `image.tag`)

**Procedimento de Manutenção (Atualização da Imagem Solr):**

1.  **Identificar Nova Tag:** A equipe responsável identifica uma nova versão/tag da imagem Solr que foi testada e aprovada para atualização (ex: `9.8.0-debian-12-r1`).
2.  **Modificar `values.yaml`:** O administrador edita o arquivo `apps/solr/solr/values.yaml` no repositório Git, alterando o valor de `image.tag` para a nova tag desejada.
3.  **Commit e Push:** As alterações são commitadas e enviadas para o repositório Git.
4.  **Implantação via GitOps:** O Fleet detecta a mudança no repositório e aplica a atualização ao chart Helm do Solr. Isso geralmente resulta em um rolling update dos pods Solr, onde os pods antigos são substituídos gradualmente por novos pods executando a nova versão da imagem.
5.  **Monitoramento:** Acompanhar o processo de atualização através do Rancher ou `kubectl` para garantir que os novos pods subam corretamente e que a aplicação permaneça saudável.

---


### 3.2. Autenticação do Solr

A segurança do acesso ao Solr é configurada nesta seção, garantindo que apenas usuários autorizados possam interagir com a plataforma.

-   **Autenticação Habilitada (`auth.enabled`):**
    -   **Valor:** `true`
    -   **Descrição Detalhada:** Ativa o módulo de autenticação básica do Solr. Quando habilitado, o Solr exigirá credenciais (usuário e senha) para acesso à sua interface de administração e APIs.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `auth.enabled`)

-   **Segredo Existente para Credenciais (`auth.existingSecret`):**
    -   **Valor:** `solr-usr-secret`
    -   **Descrição Detalhada:** Em vez de definir o usuário e senha diretamente no `values.yaml` (o que seria uma prática insegura), a configuração aponta para um Segredo Kubernetes existente chamado `solr-usr-secret`. Este segredo contém as credenciais reais.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `auth.existingSecret`)

-   **Chave da Senha no Segredo (`auth.existingSecretPasswordKey`):**
    -   **Valor:** `solr-password`
    -   **Descrição Detalhada:** Especifica qual chave dentro do Segredo `solr-usr-secret` contém o valor da senha do administrador do Solr. O nome de usuário é tipicamente `solr` ou `admin` por padrão quando a autenticação é gerenciada desta forma pelo chart Helm (o usuário exato pode depender da implementação do chart ou da configuração inicial do Solr). O Segredo `solr-usr-secret` é, por sua vez, gerenciado e populado pelo External Secrets Operator, que busca as credenciais do HashiCorp Vault, conforme detalhado na Seção 7.2.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `auth.existingSecretPasswordKey`)

**Implicações de Segurança:**
Esta abordagem de usar um `existingSecret` preenchido por um sistema externo como o Vault é uma excelente prática de segurança. Ela evita que senhas sejam armazenadas em texto plano no repositório Git e centraliza o gerenciamento de segredos.

**Procedimento de Manutenção (Rotação de Senha do Solr):**

1.  **Atualizar no Vault:** A nova senha do Solr é primeiramente atualizada no HashiCorp Vault, no caminho e chave corretos que o External Secret `solr-external-secret` está configurado para monitorar (ver Seção 7.2.3).
2.  **Sincronização Automática:** O External Secrets Operator detecta a mudança no Vault (ou em seu próximo intervalo de sincronização) e atualiza automaticamente o Segredo Kubernetes `solr-usr-secret` com a nova senha na chave `solr-password`.
3.  **Reinício dos Pods Solr (Recomendado):** Embora alguns sistemas possam recarregar configurações dinamicamente, para garantir que o Solr utilize a nova senha para todas as suas operações internas e autenticação, um reinício dos pods Solr (rolling update) é geralmente recomendado. Isso força o Solr a reler as credenciais do segredo atualizado.

---


### 3.3. Configurações Java e Otimizações do SOLR

Esta seção detalha as configurações da JVM (Java Virtual Machine) para o Solr e outros argumentos e variáveis de ambiente que afetam seu desempenho e comportamento.

-   **Memória Java (`javaMem`):**
    -   **Valor:** `"-Xms16g -Xmx100g"`
    -   **Descrição Detalhada:** Define os parâmetros de heap da JVM para os pods Solr.
        -   `-Xms16g`: Define o tamanho inicial do heap (Initial Heap Size) para 16 Gigabytes. Alocar um tamanho inicial considerável pode melhorar o desempenho de inicialização, evitando realocações frequentes de heap no início.
        -   `-Xmx100g`: Define o tamanho máximo do heap (Maximum Heap Size) para 100 Gigabytes. Este é um valor muito alto e implica que os nós Kubernetes onde o Solr roda devem ter uma quantidade substancial de memória disponível. É crucial que este valor não exceda a memória alocável no nó, descontando o que é necessário para o sistema operacional e outros pods. A solicitação de memória do pod (requests.memory) é de `20Gi`, o que significa que o Kubernetes garante 20Gi para o pod, mas o Solr pode tentar usar até 100Gi de heap. Isso pode levar a problemas de OOMKilled (Out Of Memory Killed) se o nó não tiver memória suficiente ou se os limites de memória do pod forem mais restritivos e não alinhados com `-Xmx`.
    -   **Considerações:** A alocação de memória para o Solr é crítica. Um heap muito pequeno pode levar a `OutOfMemoryError` e baixo desempenho, enquanto um heap excessivamente grande pode causar longas pausas de Garbage Collection (GC) e também impactar o desempenho. O valor de `100g` para `-Xmx` é significativo e deve ser cuidadosamente avaliado em relação aos recursos disponíveis e ao perfil de uso do Solr.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `javaMem`)

-   **Argumentos Adicionais (`args`):**
    -   **Valor:** `["-Dsolr.sharedLib","/solr-plugins/"]`
    -   **Descrição Detalhada:** Passa argumentos adicionais para a JVM ao iniciar o Solr.
        -   `-Dsolr.sharedLib=/solr-plugins/`: Esta propriedade de sistema informa ao Solr para carregar bibliotecas (JARs de plugins) do diretório `/solr-plugins/` dentro do contêiner. Este diretório é montado a partir de um volume persistente (PVC `solr-plugin-folder`), permitindo a adição de plugins customizados sem reconstruir a imagem do Solr. Ver Seção 3.6 e 7.3 para mais detalhes sobre a montagem e o servidor de upload de plugins.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `args`)

-   **Variáveis de Ambiente Extras (`extraEnvVars`):**
    -   **`SOLR_OPTS`:**
        -   **Valor:** `"-Dsolr.sharedLib=/solr-plugins/ -Dsolr.allowPaths=* -Dsolr.disable.allowUrls=true -Dsolr.cloud.client.stallTime=90000 -Djute.maxbuffer=5000000 -Dsolr.max.booleanClauses=10000 -javaagent:/opt/newrelic/newrelic.jar"`
        -   **Descrição Detalhada:** Esta variável de ambiente é usada para passar um conjunto de opções de sistema Java (`-D`) e outras configurações para o Solr.
            -   `-Dsolr.sharedLib=/solr-plugins/`: Redundante com a configuração em `args`, mas inofensivo.
            -   `-Dsolr.allowPaths=*`: Permite que o Solr acesse qualquer caminho no sistema de arquivos para certas operações (como backup/restore em locais arbitrários, se não restrito de outra forma). Embora flexível, usar `*` pode ter implicações de segurança se não for cuidadosamente gerenciado em conjunto com a política de segurança Java (Seção 6). Idealmente, caminhos específicos deveriam ser listados.
            -   `-Dsolr.disable.allowUrls=true`: Desabilita a funcionalidade `allowUrls` do Solr, que poderia ser usada para buscar recursos de URLs arbitrárias, potencialmente levando a vulnerabilidades de SSRF (Server-Side Request Forgery). Desabilitá-la é uma boa prática de segurança.
            -   `-Dsolr.cloud.client.stallTime=90000`: Define o tempo máximo (em milissegundos, aqui 90 segundos) que um cliente SolrCloud esperará por uma resposta de um nó antes de considerá-lo parado ou lento. Aumentar este valor pode ajudar em redes com maior latência ou durante operações pesadas, mas também pode atrasar a detecção de nós realmente problemáticos.
            -   `-Djute.maxbuffer=5000000`: Define o tamanho máximo do buffer (aproximadamente 5MB) para mensagens Jute, o protocolo de serialização usado pelo Zookeeper. Aumentar este valor é necessário se o Solr estiver gerenciando configurações muito grandes ou um grande número de collections/shards, cujas informações são armazenadas no Zookeeper. O valor padrão é geralmente menor (1MB).
            -   `-Dsolr.max.booleanClauses=10000`: Aumenta o número máximo de cláusulas booleanas permitidas em uma consulta Lucene/Solr. O padrão é 1024. Consultas muito complexas ou geradas programaticamente podem exceder o limite padrão. Aumentar este limite permite tais consultas, mas pode ter implicações de desempenho se consultas excessivamente complexas forem frequentes.
            -   `-javaagent:/opt/newrelic/newrelic.jar`: Configura o agente APM (Application Performance Monitoring) da New Relic. O JAR do agente é carregado na inicialização da JVM e instrumenta o código do Solr para coletar métricas de desempenho e enviá-las para a plataforma New Relic. O diretório `/opt/newrelic/` é montado a partir de um volume NFS (ver Seção 3.6).
    -   **`SOLR_LOG_LEVEL`:**
        -   **Valor:** `"WARN"`
        -   **Descrição Detalhada:** Define o nível de log padrão para o Solr. As opções comuns incluem `INFO`, `WARN`, `ERROR`, `DEBUG`. `WARN` significa que apenas mensagens de aviso, erro e fatais serão registradas por padrão, reduzindo a verbosidade dos logs em comparação com `INFO` ou `DEBUG`. Isso é geralmente apropriado para ambientes de produção.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `extraEnvVars`)

**Procedimento de Manutenção (Ajuste de Configurações Java/SOLR_OPTS):**

1.  **Análise e Decisão:** Com base em observações de desempenho, logs de erro, ou novas recomendações, a equipe decide ajustar um parâmetro (ex: aumentar `Djute.maxbuffer` ou alterar `SOLR_LOG_LEVEL`).
2.  **Modificar `values.yaml`:** O administrador edita o arquivo `apps/solr/solr/values.yaml` no repositório Git, alterando o valor da propriedade desejada em `javaMem`, `args` ou `extraEnvVars`.
3.  **Commit e Push:** As alterações são versionadas no Git.
4.  **Implantação via GitOps:** O Fleet aplica a atualização, o que resultará em um rolling update dos pods Solr para que as novas configurações da JVM ou variáveis de ambiente entrem em vigor.
5.  **Monitoramento:** Verificar os logs do Solr e o comportamento da aplicação para confirmar que a alteração teve o efeito desejado e não introduziu problemas.

---


### 3.4. Gerenciamento de Réplicas, Agendamento de Pods e Afinidade

Esta seção do `values.yaml` controla quantos pods Solr serão executados, em quais nós do cluster Kubernetes eles podem ser agendados, e como eles devem ser distribuídos para alta disponibilidade.

-   **Contagem de Réplicas (`replicaCount`):**
    -   **Valor:** `5`
    -   **Descrição Detalhada:** Define o número desejado de pods Solr que o StatefulSet (ou Deployment, dependendo do chart) tentará manter em execução. Ter múltiplas réplicas é essencial para alta disponibilidade e para distribuir a carga de indexação e busca. Com 5 réplicas, o SolrCloud pode tolerar a falha de alguns pods sem interrupção do serviço, assumindo que as collections estão configuradas com fatores de replicação adequados.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `replicaCount`)

-   **Seletor de Nó (`nodeSelector`):**
    -   **Valor:** `solr: "true"`
    -   **Descrição Detalhada:** Restringe o agendamento dos pods Solr apenas para nós Kubernetes que possuem o rótulo (label) `solr: "true"`. Isso permite dedicar um conjunto específico de nós (workers) para a carga de trabalho do Solr, que pode ter requisitos de recursos diferentes (ex: mais CPU, memória, ou discos rápidos) do restante do cluster.
    -   **Procedimento Prático:** Para que isso funcione, os administradores do cluster Kubernetes devem rotular os nós apropriados com `kubectl label node <nome-do-no> solr=true`.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `nodeSelector`)

-   **Tolerâncias (`tolerations`):**
    -   **Valor:** `effect: NoSchedule, key: solr, value: "true", operator: Equal`
    -   **Descrição Detalhada:** Permite que os pods Solr sejam agendados em nós que possuem um "taint" (marcação) específico. Taints são usados para repelir pods de certos nós. Se os nós dedicados ao Solr (com o rótulo `solr: "true"`) também tiverem um taint como `solr=true:NoSchedule` (para impedir que outros pods, sem esta tolerância, sejam agendados neles), esta tolerância garante que os pods Solr possam, de fato, ser agendados nesses nós dedicados.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `tolerations`)

-   **Rótulos do Pod (`podLabels`):**
    -   **Valor:** `app: solr`
    -   **Descrição Detalhada:** Adiciona o rótulo `app: solr` a todos os pods Solr criados. Estes rótulos são usados internamente pelo Kubernetes para selecionar e gerenciar grupos de pods (ex: por Services, Deployments, StatefulSets) e também para definir regras de afinidade/anti-afinidade.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `podLabels`)

-   **Afinidade (`affinity`):**
    -   **Valor:** `podAntiAffinity` configurado com `requiredDuringSchedulingIgnoredDuringExecution` para `app: solr` e `topologyKey: "kubernetes.io/hostname"`.
    -   **Descrição Detalhada:** Esta configuração de anti-afinidade de pod instrui o scheduler do Kubernetes a *tentar fortemente* (devido a `requiredDuringSchedulingIgnoredDuringExecution`) não agendar múltiplos pods Solr (identificados pelo rótulo `app: solr`) no mesmo nó físico ou virtual (identificado pela `topologyKey: "kubernetes.io/hostname"`).
        -   `requiredDuringSchedulingIgnoredDuringExecution`: A regra de anti-afinidade *deve* ser satisfeita durante o agendamento. Se não puder ser satisfeita (ex: não há nós suficientes para espalhar os pods), o pod não será agendado. Uma vez que o pod está agendado, se as condições de afinidade mudarem (ex: rótulos de outros pods), o pod não será removido.
        -   `labelSelector.matchExpressions`: Define que a regra se aplica a pods com o rótulo `app: solr`.
        -   `topologyKey: "kubernetes.io/hostname"`: Define o domínio da topologia. `kubernetes.io/hostname` significa que a anti-afinidade é aplicada no nível do nó individual. O objetivo é distribuir os pods Solr por diferentes nós para aumentar a resiliência. Se um nó falhar, apenas os pods Solr naquele nó serão afetados, e as outras réplicas em outros nós continuarão operando.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `affinity`)

**Implicações para Alta Disponibilidade:**
A combinação de `replicaCount`, `nodeSelector`, `tolerations`, e `podAntiAffinity` é crucial para alcançar uma implantação Solr altamente disponível e resiliente. Ela garante que haja múltiplas instâncias, que elas rodem em hardware apropriado e que sejam distribuídas para minimizar o impacto de falhas de nó.

---


### 3.5. Alocação de Recursos Computacionais (Limits e Requests)

A definição adequada de solicitações (requests) e limites (limits) de recursos (CPU e memória) para os pods Solr é fundamental para garantir o desempenho estável da aplicação e a utilização eficiente dos recursos do cluster Kubernetes.

-   **Recursos para o Contêiner Principal do Solr (`resources`):**
    -   **Limites (`resources.limits`):**
        -   **Valor:** `{}` (Vazio, ou seja, não há limites de CPU ou memória explicitamente definidos para o contêiner Solr neste nível).
        -   **Descrição Detalhada:** Quando os limites não são definidos no manifesto do pod, o comportamento depende da configuração do namespace ou do cluster. Se houver `LimitRange` definido no namespace, ele pode aplicar limites padrão. Caso contrário, o pod Solr pode, teoricamente, consumir todos os recursos de CPU disponíveis no nó ou uma grande quantidade de memória até o limite do nó ou ser terminado pelo OOM killer do sistema operacional se exceder a memória do nó. No entanto, a configuração de `javaMem` com `-Xmx100g` (Seção 3.3) atua como um limite de heap para a JVM, mas não é um limite no nível do contêiner que o Kubernetes gerencia diretamente para fins de agendamento ou terminação por excesso de uso de memória do *contêiner* (que inclui heap + off-heap da JVM + outros processos no contêiner).
        -   **Consideração Importante:** É uma prática recomendada definir limites de memória para evitar que um pod consuma excessivamente os recursos do nó e afete outros pods. A ausência de um limite de memória no contêiner, especialmente com um `-Xmx` tão alto, é um risco. O limite de CPU, se não definido, significa que o pod pode usar toda a CPU disponível, o que pode ser desejável para uma carga de trabalho intensiva como o Solr, mas também pode levar à contenção de CPU se outros pods no mesmo nó também forem intensivos em CPU.

    -   **Requisições (`resources.requests`):**
        -   **CPU (`resources.requests.cpu`):** `"1"` (Equivalente a 1 core de CPU ou 1000m - millicores).
            -   **Descrição Detalhada:** Garante que o pod Solr terá 1 core de CPU reservado para ele no nó onde for agendado. O Kubernetes usará essa informação para decisões de agendamento, garantindo que o pod seja colocado em um nó com pelo menos 1 CPU disponível.
        -   **Memória (`resources.requests.memory`):** `20Gi` (20 Gibibytes).
            -   **Descrição Detalhada:** Garante que o pod Solr terá 20Gi de memória reservados no nó. Similar à CPU, isso é usado para agendamento. O pod terá essa quantidade de memória garantida. Se o pod usar mais do que o solicitado e menos que o limite (se definido), ele é classificado como "Burstable". Se usar mais que o limite, pode ser terminado (OOMKilled pelo Kubernetes se for limite de memória).
        -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `resources`)

**Qualidade de Serviço (QoS Class):**
Com `requests` definidos e `limits` não definidos (ou definidos com valores diferentes dos requests), o pod Solr provavelmente terá uma QoS Class de **Burstable**. Se os limites fossem definidos e iguais aos requests, a QoS Class seria **Guaranteed**.

-   **Burstable:** Os pods têm alguma garantia de recursos (requests), mas podem usar mais, até os limites (se definidos) ou os recursos do nó. Eles têm uma prioridade de terminação média se o nó ficar sem recursos.

**Relação com `javaMem`:**
É crucial que `resources.requests.memory` (20Gi) e `resources.limits.memory` (se definido) sejam consistentes com `javaMem` (`-Xms16g -Xmx100g`).
-   O `requests.memory` de 20Gi é maior que o `-Xms16g`, o que é bom, pois a JVM precisa de memória para o heap inicial mais o overhead (off-heap, threads, etc.).
-   No entanto, o `-Xmx100g` permite que a JVM tente alocar um heap muito maior que os 20Gi solicitados. Se o pod Solr tentar usar perto de 100Gi de memória (heap + off-heap), e não houver um limite de contêiner correspondente, ele dependerá da memória disponível no nó. Se um limite de contêiner fosse definido (ex: 30Gi), e o Solr tentasse usar 100Gi de heap, ele seria OOMKilled pelo Kubernetes muito antes de atingir o `-Xmx` da JVM.
-   A ausência de um limite de memória no contêiner, com um `-Xmx` tão alto, significa que o Solr pode, em teoria, tentar usar até 100Gi de heap mais memória off-heap, o que exigiria nós com uma quantidade massiva de RAM. Se os nós não tiverem essa capacidade, ou se outros pods competirem por memória, os pods Solr correm alto risco de serem OOMKilled pelo sistema operacional.

**Procedimento de Manutenção (Ajuste de Recursos):**

1.  **Monitoramento:** Observar o uso real de CPU e memória dos pods Solr usando ferramentas como o Prometheus/Grafana (configurado na Seção 3.10) ou a interface do Rancher.
2.  **Análise:** Determinar se os requests são adequados (Solr está recebendo os recursos que precisa para operar bem) e se os limites (ou a ausência deles) estão causando problemas (ex: OOMKilled, contenção de CPU no nó).
3.  **Modificar `values.yaml`:** Ajustar os valores de `resources.requests.cpu`, `resources.requests.memory`, e considerar adicionar `resources.limits.cpu` e `resources.limits.memory` de forma consistente com o `-Xmx` e o comportamento observado.
4.  **Commit e Push:** Salvar as alterações no Git.
5.  **Implantação via GitOps:** O Fleet aplicará as mudanças, o que pode recriar os pods Solr com as novas configurações de recursos.
6.  **Re-monitoramento:** Observar o comportamento com os novos ajustes.

---


### 3.6. Configuração de Volumes de Dados e Montagens de Arquivos Essenciais

Esta seção descreve como os volumes persistentes e outros tipos de volumes (como segredos e NFS para backups/plugins) são definidos e montados nos contêineres Solr. O armazenamento adequado é vital para a persistência dos dados do Solr, o carregamento de plugins, a execução de backups e a aplicação de configurações de segurança.

-   **Volumes Extras (`extraVolumes`):**
    Esta matriz define volumes adicionais que estarão disponíveis para os pods Solr, além do volume principal de persistência de dados (discutido na Seção 3.9).

    -   **Volume para Plugins (`plugin-dir`):**
        -   **Tipo:** `persistentVolumeClaim`
        -   **Nome do PVC (`claimName`):** `solr-plugin-folder`
        -   **Descrição Detalhada:** Define um volume chamado `plugin-dir` que utiliza um PersistentVolumeClaim (PVC) existente chamado `solr-plugin-folder`. Este PVC é usado para armazenar JARs de plugins Solr customizados. O PVC `solr-plugin-folder` é provavelmente provisionado via NFS com modo de acesso `ReadWriteMany` para ser compartilhado entre o servidor de upload de plugins (Seção 7.3) e todos os pods Solr.
        -   **Referência GitOps:** `apps/solr/solr/values.yaml` (dentro de `extraVolumes`)

    -   **Volume para Backups (`backup-nfs`):**
        -   **Tipo:** `nfs`
        -   **Caminho no Servidor NFS (`path`):** `/nas-solr-bkp/producao/`
        -   **Servidor NFS (`server`):** `nas3.tcu.gov.br`
        -   **Descrição Detalhada:** Define um volume chamado `backup-nfs` que monta um compartilhamento NFS diretamente. Este volume é usado para armazenar os backups do Solr. O servidor `nas3.tcu.gov.br` é o host do servidor NFS, e `/nas-solr-bkp/producao/` é o caminho exportado nesse servidor.
        -   **Referência GitOps:** `apps/solr/solr/values.yaml` (dentro de `extraVolumes`)

    -   **Volume para New Relic (`newrelic-nfs`):**
        -   **Tipo:** `nfs`
        -   **Caminho no Servidor NFS (`path`):** `/nas-solr-bkp/newrelic/`
        -   **Servidor NFS (`server`):** `nas3.tcu.gov.br`
        -   **Descrição Detalhada:** Define um volume chamado `newrelic-nfs`, também montado do mesmo servidor NFS `nas3.tcu.gov.br`, mas de um caminho diferente (`/nas-solr-bkp/newrelic/`). Este volume é usado para armazenar arquivos relacionados ao agente New Relic, como sua configuração ou o próprio JAR do agente, se não estiver embutido na imagem Solr.
        -   **Referência GitOps:** `apps/solr/solr/values.yaml` (dentro de `extraVolumes`)

    -   **Volume para Política de Segurança (`solr-policy`):**
        -   **Tipo:** `secret`
        -   **Nome do Segredo (`secretName`):** `solr-policy`
        -   **Descrição Detalhada:** Define um volume chamado `solr-policy` que é populado a partir de um Segredo Kubernetes chamado `solr-policy`. Este segredo contém o arquivo `security.policy` da Java Security Manager, que define permissões granulares para a JVM do Solr. O segredo `solr-policy` é, por sua vez, gerenciado pelo Kustomize, conforme detalhado na Seção 5.
        -   **Referência GitOps:** `apps/solr/solr/values.yaml` (dentro de `extraVolumes`)

-   **Montagens de Volume Extras (`extraVolumeMounts`):**
    Esta matriz especifica como os `extraVolumes` definidos acima são montados dentro do(s) contêiner(es) Solr.

    -   **Montagem para Backups:**
        -   **Caminho de Montagem (`mountPath`):** `/bitnami/backups/`
        -   **Nome do Volume (`name`):** `backup-nfs`
        -   **Descrição Detalhada:** Monta o volume NFS `backup-nfs` no caminho `/bitnami/backups/` dentro dos pods Solr. É neste diretório que o Solr salvará seus snapshots de backup quando a API de Backup for acionada (ver Seção 7.1).
        -   **Referência GitOps:** `apps/solr/solr/values.yaml` (dentro de `extraVolumeMounts`)

    -   **Montagem para Plugins:**
        -   **Caminho de Montagem (`mountPath`):** `/solr-plugins`
        -   **Nome do Volume (`name`):** `plugin-dir`
        -   **Descrição Detalhada:** Monta o volume `plugin-dir` (que é o PVC `solr-plugin-folder`) no caminho `/solr-plugins/`. O Solr é configurado (via `-Dsolr.sharedLib=/solr-plugins/`, Seção 3.3) para carregar plugins deste diretório.
        -   **Referência GitOps:** `apps/solr/solr/values.yaml` (dentro de `extraVolumeMounts`)

    -   **Montagem para New Relic:**
        -   **Caminho de Montagem (`mountPath`):** `/opt/newrelic/`
        -   **Nome do Volume (`name`):** `newrelic-nfs`
        -   **Descrição Detalhada:** Monta o volume NFS `newrelic-nfs` no caminho `/opt/newrelic/`. O agente Java da New Relic (`-javaagent:/opt/newrelic/newrelic.jar`, Seção 3.3) espera encontrar seus arquivos de configuração e o JAR do agente neste local.
        -   **Referência GitOps:** `apps/solr/solr/values.yaml` (dentro de `extraVolumeMounts`)

    -   **Montagem para Política de Segurança Java:**
        -   **Caminho de Montagem (`mountPath`):** `/opt/bitnami/solr/server/etc/security.policy`
        -   **Nome do Volume (`name`):** `solr-policy`
        -   **Sub Caminho (`subPath`):** `security.policy`
        -   **Descrição Detalhada:** Monta o arquivo `security.policy` (que é a chave dentro do Segredo `solr-policy`) diretamente no caminho `/opt/bitnami/solr/server/etc/security.policy` dentro do contêiner Solr. O uso de `subPath` permite montar um arquivo específico de um segredo em vez do diretório inteiro. Este caminho é onde o Solr espera encontrar seu arquivo de política de segurança Java, sobrescrevendo qualquer política padrão que possa existir na imagem.
        -   **Referência GitOps:** `apps/solr/solr/values.yaml` (dentro de `extraVolumeMounts`)

**Importância da Configuração de Volumes:**
Uma configuração correta de volumes e montagens é essencial para:
-   **Persistência de Dados:** Embora o volume principal de dados do Solr seja definido em `persistence` (Seção 3.9), estes volumes extras são cruciais para funcionalidades adicionais.
-   **Extensibilidade:** Permitir o carregamento de plugins customizados.
-   **Operações de Manutenção:** Facilitar backups e restaurações.
-   **Segurança:** Aplicar políticas de segurança customizadas.
-   **Monitoramento:** Habilitar agentes APM como o New Relic.

---


### 3.7. Contêineres de Inicialização (Init Containers) para Preparação de Ambiente

Init Containers são contêineres especializados que rodam antes dos contêineres principais da aplicação em um Pod. Eles são usados para executar tarefas de setup ou pré-condições necessárias para que a aplicação principal funcione corretamente. No caso do Solr, um Init Container é usado para ajustar permissões em volumes montados.

-   **Configuração do Init Container (`initContainers`):**
    -   **Nome (`name`):** `init-volume-perm`
        -   **Descrição Detalhada:** Um nome descritivo para o Init Container, indicando sua função de inicializar permissões de volume.
    -   **Imagem (`image`):** `bitnami/os-shell`
        -   **Descrição Detalhada:** Utiliza uma imagem Docker leve (`bitnami/os-shell`) que contém um shell básico e utilitários comuns do sistema operacional, suficiente para executar comandos como `chown`.
    -   **Política de Pull da Imagem (`imagePullPolicy`):** `IfNotPresent`
        -   **Descrição Detalhada:** O Kubernetes tentará baixar a imagem apenas se ela não estiver já presente localmente no nó.
    -   **Contexto de Segurança (`securityContext.runAsUser`):** `0`
        -   **Descrição Detalhada:** Executa o Init Container como o usuário root (UID 0). Isso é necessário porque o comando `chown` (change owner) requer privilégios de root para alterar a propriedade de arquivos e diretórios que podem pertencer a outros usuários.
    -   **Comando (`command`):**
        -   **Valor:** `["sh", "-c", "chown -R 1001:1001 /bitnami/backups/ & chown -R 1001:1001 /opt/newrelic/"]`
        -   **Descrição Detalhada:** Executa um comando shell.
            -   `chown -R 1001:1001 /bitnami/backups/`: Altera recursivamente (`-R`) o proprietário e o grupo do diretório `/bitnami/backups/` (montado do NFS para backups) para o usuário com UID 1001 e GID 1001. O UID 1001 é frequentemente o usuário não-root padrão dentro de muitas imagens de contêiner, incluindo as imagens Solr da Bitnami. Garantir que o processo Solr (rodando como UID 1001) tenha permissão de escrita neste diretório é essencial para que os backups funcionem.
            -   `& chown -R 1001:1001 /opt/newrelic/`: Similarmente, altera as permissões do diretório `/opt/newrelic/` (montado do NFS para o agente New Relic) para o usuário 1001. Isso permite que o agente New Relic, ou o processo Solr, escreva logs ou outros arquivos necessários neste diretório.
    -   **Montagens de Volume (`volumeMounts`):**
        -   Monta o volume `backup-nfs` em `/bitnami/backups/`.
        -   Monta o volume `newrelic-nfs` em `/opt/newrelic/`.
        -   **Descrição Detalhada:** Estas montagens garantem que os comandos `chown` dentro do Init Container operem nos volumes NFS corretos que serão posteriormente usados pelo contêiner principal do Solr.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `initContainers`)

**Funcionamento e Importância:**
O Init Container `init-volume-perm` executa e completa sua tarefa (ajustar permissões) *antes* que o contêiner principal do Solr seja iniciado. Se o Init Container falhar (ex: o comando `chown` falhar), o Pod não prosseguirá para iniciar o contêiner Solr, e o Pod será marcado como falho ou entrará em um loop de reinicialização (CrashLoopBackOff) dependendo da política de reinício do Pod.
Este passo é crucial, especialmente ao usar volumes NFS, onde as permissões de UID/GID no servidor NFS podem não corresponder automaticamente ao UID/GID do processo dentro do contêiner. O Init Container resolve essa incompatibilidade de permissões programaticamente na inicialização do Pod.

---


### 3.8. Configuração de Acesso Externo (Ingress)

O Ingress no Kubernetes gerencia o acesso externo aos serviços dentro do cluster, tipicamente para tráfego HTTP e HTTPS. Esta seção do `values.yaml` define como o Solr será exposto externamente.

-   **Ingress Habilitado (`ingress.enabled`):**
    -   **Valor:** `true`
    -   **Descrição Detalhada:** Cria um recurso Ingress para o serviço Solr, tornando-o acessível de fora do cluster Kubernetes.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `ingress.enabled`)

-   **Hostname (`ingress.hostname`):**
    -   **Valor:** `solr.producao.rancher.tcu.gov.br`
    -   **Descrição Detalhada:** Define o nome de host (URL) que será usado para acessar a interface de administração e as APIs do Solr externamente. O tráfego direcionado a este hostname será roteado pelo Ingress Controller para o serviço Solr.
    -   **Procedimento Prático:** É necessário que uma entrada DNS correspondente a `solr.producao.rancher.tcu.gov.br` seja criada e aponte para o endereço IP do Ingress Controller do cluster Rancher.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `ingress.hostname`)

-   **Caminho (`ingress.path`):**
    -   **Valor:** `/`
    -   **Descrição Detalhada:** Define o caminho base na URL para o qual o tráfego será roteado para o Solr. Usar `/` significa que todo o tráfego para o hostname `solr.producao.rancher.tcu.gov.br` será direcionado ao Solr.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `ingress.path`)

-   **Anotações (`ingress.annotations`):**
    -   **Valor:** `{}` (Vazio)
    -   **Descrição Detalhada:** Anotações são usadas para configurar comportamentos específicos do Ingress Controller (ex: Nginx Ingress, Traefik). A ausência de anotações aqui significa que o comportamento padrão do Ingress Controller do cluster será aplicado, ou que configurações globais de Ingress já suprem as necessidades. Anotações comuns podem incluir configurações de reescrita de URL, limites de tamanho de corpo de requisição, timeouts, autenticação externa, etc.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `ingress.annotations`)

-   **TLS (`ingress.tls`):**
    -   **Valor:** `false`
    -   **Descrição Detalhada:** Indica que este recurso Ingress específico não gerenciará a terminação TLS (HTTPS). Isso não significa necessariamente que o acesso não é seguro. É comum que a terminação TLS seja tratada em um nível superior, como por um balanceador de carga externo à frente do cluster Kubernetes, ou por um Ingress Controller global que aplica TLS a todos os hostnames gerenciados. Se o tráfego entre o balanceador de carga/Ingress Controller e os pods Solr não for criptografado, pode haver um risco de segurança na rede interna do cluster.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `ingress.tls`)

-   **Auto-assinado (`ingress.selfSigned`):**
    -   **Valor:** `false`
    -   **Descrição Detalhada:** Relevante apenas se `ingress.tls` fosse `true` e o chart estivesse configurado para gerar um certificado autoassinado para TLS. Como `ingress.tls` é `false`, esta configuração não tem efeito prático aqui.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `ingress.selfSigned`)

**Fluxo de Acesso Externo:**

1.  O usuário acessa `http://solr.producao.rancher.tcu.gov.br/` (ou `https` se o TLS for tratado externamente).
2.  A requisição DNS resolve para o IP do Ingress Controller do cluster.
3.  O Ingress Controller recebe a requisição, verifica as regras de Ingress e, com base no hostname e caminho, encaminha o tráfego para o Service Kubernetes do Solr.
4.  O Service do Solr, por sua vez, balanceia a carga entre os pods Solr disponíveis.

---


### 3.9. Persistência de Dados do SOLR (Collections e Índices)

A persistência de dados é crítica para o Solr, pois é onde os índices das collections são armazenados. Se os dados não forem persistidos corretamente, uma reinicialização de pod resultaria na perda de todos os dados indexados. Esta seção do `values.yaml` configura como os volumes persistentes são provisionados para cada pod Solr.

-   **Persistência Habilitada (`persistence.enabled`):**
    -   **Valor:** `true`
    -   **Descrição Detalhada:** Habilita o uso de PersistentVolumeClaims (PVCs) para armazenar os dados do Solr. Quando `true`, o chart Helm criará um PVC para cada réplica do Solr (se estiver usando um StatefulSet, que é o padrão para aplicações com estado como SolrCloud).
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `persistence.enabled`)

-   **Política de Recuperação (`persistence.reclaimPolicy`):**
    -   **Valor:** `Retain`
    -   **Descrição Detalhada:** Define a política de recuperação para os PersistentVolumes (PVs) que são provisionados para os PVCs do Solr. `Retain` significa que, se um PVC for excluído, o PV subjacente (e, portanto, os dados nele) não será excluído automaticamente. Ele permanecerá no cluster e precisará ser recuperado ou excluído manualmente. Isso é uma medida de segurança importante para evitar a perda acidental de dados. As outras opções comuns são `Delete` (o PV é excluído com o PVC) e `Recycle` (depreciado).
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `persistence.reclaimPolicy`)

-   **Modos de Acesso (`persistence.accessModes`):**
    -   **Valor:** `[ReadWriteOnce]`
    -   **Descrição Detalhada:** Especifica como o volume pode ser montado pelos nós.
        -   `ReadWriteOnce` (RWO): O volume pode ser montado como leitura-escrita por um único nó. Isso é adequado para StatefulSets, onde cada pod geralmente obtém seu próprio PVC e o volume associado é montado exclusivamente naquele pod/nó. Para SolrCloud, cada nó Solr gerencia seus próprios shards ou réplicas de shards, e RWO é o modo de acesso apropriado para os volumes de dados individuais de cada pod.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `persistence.accessModes`)

-   **Tamanho (`persistence.size`):**
    -   **Valor:** `1000Gi`
    -   **Descrição Detalhada:** Define o tamanho solicitado para cada PersistentVolumeClaim criado para os pods Solr. Cada réplica do Solr solicitará um volume de 1000 Gibibytes. Este é um tamanho considerável e deve ser planejado com base na quantidade de dados esperada para indexação e no crescimento futuro.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `persistence.size`)

-   **Caminho de Montagem (`persistence.mountPath`):**
    -   **Valor:** `/bitnami/solr`
    -   **Descrição Detalhada:** Especifica o caminho dentro do contêiner Solr onde o volume persistente será montado. A imagem Solr da Bitnami (e muitas outras) espera que seus dados (índices, configurações de core, etc.) residam neste diretório ou em um subdiretório dele.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `persistence.mountPath`)

-   **Classe de Armazenamento (`persistence.storageClass`):**
    -   **Valor:** `sc-solr`
    -   **Descrição Detalhada:** Especifica o nome da `StorageClass` a ser usada para provisionar dinamicamente os PersistentVolumes. Uma `StorageClass` define o tipo de armazenamento subjacente (ex: SSD rápido, disco magnético mais lento, armazenamento vSphere específico, etc.) e seus parâmetros. O uso de uma `StorageClass` customizada como `sc-solr` sugere que há uma configuração de armazenamento específica e otimizada para a carga de trabalho do Solr no ambiente vSphere do TCU. Esta `StorageClass` é definida em `apps/vsphere/sc-solr/` (ver Seção 8).
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `persistence.storageClass`)

**Funcionamento com StatefulSets:**
O Solr, especialmente em modo SolrCloud, é uma aplicação com estado. Charts Helm para Solr geralmente usam StatefulSets para gerenciar os pods Solr. StatefulSets fornecem identidades de rede estáveis e armazenamento persistente estável para cada pod.
-   Cada pod em um StatefulSet recebe um nome ordinal estável (ex: `solr-0`, `solr-1`, `solr-2`, ...).
-   Para cada pod, um PVC correspondente é criado (ex: `data-solr-0`, `data-solr-1`, ...), usando a `storageClass` e o `size` definidos.
-   Estes PVCs são então vinculados a PVs provisionados dinamicamente (ou estaticamente, se disponíveis).

**Procedimento de Manutenção (Expansão de Volume):**
Se os volumes de dados do Solr estiverem ficando cheios, pode ser necessário expandi-los.
1.  **Verificar Capacidade da StorageClass:** Primeiro, verificar se a `StorageClass` `sc-solr` suporta expansão de volume online (`allowVolumeExpansion: true` na definição da StorageClass).
2.  **Editar PVCs (se suportado):** Se a expansão for suportada, o tamanho dos PVCs existentes pode ser editado (aumentado). No entanto, a alteração direta de `persistence.size` no `values.yaml` e a reaplicação do chart Helm podem não expandir automaticamente os PVCs existentes para StatefulSets, dependendo da versão do Helm e do Kubernetes e da configuração do chart. Muitas vezes, a expansão de PVCs para StatefulSets precisa ser feita manualmente (editando cada PVC) ou através de procedimentos específicos.
3.  **Monitoramento:** Após a tentativa de expansão, monitorar os eventos do PVC e do PV para garantir que a expansão foi bem-sucedida no nível do armazenamento e que o sistema de arquivos dentro do contêiner Solr reflete o novo tamanho (pode exigir uma reinicialização do pod ou comandos específicos no sistema operacional do contêiner, dependendo do tipo de volume e do sistema de arquivos).

---


### 3.10. Coleta de Métricas com Prometheus para Monitoramento

O monitoramento eficaz é essencial para manter a saúde e o desempenho do ambiente Solr. Esta seção do `values.yaml` configura a exposição de métricas do Solr para o Prometheus, um popular sistema de monitoramento e alerta de código aberto.

-   **Métricas Habilitadas (`metrics.enabled`):**
    -   **Valor:** `true`
    -   **Descrição Detalhada:** Habilita a implantação de um exportador de métricas para o Solr. Geralmente, isso envolve a execução de um contêiner sidecar (como o `solr-exporter` ou um exportador JMX genérico configurado para Solr) junto com cada pod Solr, ou um endpoint de métricas exposto diretamente pelo Solr que o Prometheus pode "raspar" (coletar).
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `metrics.enabled`)

-   **Recursos para o Contêiner de Métricas (`metrics.resources`):**
    Se um contêiner exportador de métricas separado for usado (ex: `solr-exporter`), esta seção define seus requests e limits de CPU e memória.
    -   **Limites (`metrics.resources.limits`):**
        -   CPU: `450m`
        -   Memória: `2Gi`
        -   Armazenamento Efêmero (`ephemeral-storage`): `1Gi`
    -   **Requisições (`metrics.resources.requests`):**
        -   CPU: `100m`
        -   Memória: `512Mi`
        -   Armazenamento Efêmero (`ephemeral-storage`): `50Mi`
    -   **Descrição Detalhada:** Define recursos modestos para o contêiner do exportador de métricas, garantindo que ele não consuma excessivamente os recursos do nó, mas tenha o suficiente para operar. O armazenamento efêmero é usado para logs temporários ou outros dados que o exportador possa precisar.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `metrics.resources`)

-   **ServiceMonitor (`metrics.serviceMonitor`):**
    O `ServiceMonitor` é um recurso customizado (CRD) fornecido pelo Prometheus Operator. Ele define como o Prometheus deve descobrir e raspar métricas de um conjunto de endpoints.
    -   **Habilitado (`metrics.serviceMonitor.enabled`):** `true`
        -   **Descrição Detalhada:** Cria um recurso `ServiceMonitor` para os pods Solr. Isso permite que uma instância do Prometheus gerenciada pelo Prometheus Operator descubra automaticamente os endpoints de métricas do Solr.
    -   **Namespace (`metrics.serviceMonitor.namespace`):** `solr`
        -   **Descrição Detalhada:** Especifica que o recurso `ServiceMonitor` será criado no namespace `solr`. O Prometheus Operator deve ser configurado para observar ServiceMonitors neste namespace.
    -   **Intervalo (`metrics.serviceMonitor.interval`):** `15s`
        -   **Descrição Detalhada:** Define a frequência com que o Prometheus tentará raspar as métricas dos endpoints do Solr (a cada 15 segundos).
    -   **Timeout de Coleta (`metrics.serviceMonitor.scrapeTimeout`):** `15s`
        -   **Descrição Detalhada:** Define o tempo máximo que o Prometheus esperará por uma resposta do endpoint de métricas antes de considerar a coleta como falha. É importante que o `scrapeTimeout` não seja maior que o `interval`.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `metrics.serviceMonitor`)

-   **Seletor de Nó para Métricas (`metrics.nodeSelector`):**
    -   **Valor:** `solr: "true"`
    -   **Descrição Detalhada:** Se o exportador de métricas for um pod separado (não um sidecar), este seletor garante que ele seja agendado nos mesmos nós dedicados ao Solr. Se for um sidecar, esta configuração pode não ser diretamente aplicável ao sidecar, mas ao pod principal.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `metrics.nodeSelector`)

-   **Tolerâncias para Métricas (`metrics.tolerations`):**
    -   **Valor:** `effect: NoSchedule, key: solr, value: "true", operator: Equal`
    -   **Descrição Detalhada:** Similar ao `nodeSelector`, permite que os pods do exportador de métricas (se separados) sejam agendados em nós com o taint `solr=true:NoSchedule`.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `metrics.tolerations`)

**Funcionamento do Monitoramento com Prometheus:**

1.  Os pods Solr (ou seus sidecars exportadores) expõem um endpoint HTTP (ex: `/metrics`) com as métricas no formato Prometheus.
2.  O `ServiceMonitor` criado instrui o Prometheus a encontrar esses endpoints (geralmente selecionando Services que correspondem aos pods Solr).
3.  O Prometheus raspa periodicamente esses endpoints, coleta as métricas e as armazena em seu banco de dados de séries temporais.
4.  As métricas podem então ser consultadas usando PromQL, visualizadas em dashboards (ex: Grafana) e usadas para configurar alertas sobre condições anormais (ex: alta latência de consulta, baixo número de réplicas disponíveis, alto uso de heap).

**Métricas Típicas do Solr:**
O Solr expõe uma grande variedade de métricas via JMX, que podem ser convertidas para o formato Prometheus. Estas incluem métricas sobre:
-   JVM (uso de heap, garbage collection, threads)
-   Caches do Solr (queryResultCache, filterCache, documentCache, etc. - hits, misses, evictions, tamanho)
-   Desempenho de Query Handlers (requisições por segundo, erros, latência média/percentil)
-   Operações de atualização (commits, otimizações)
-   Status do SolrCloud (número de nós, collections, shards, réplicas)

---


### 3.11. Configuração do Zookeeper para Operação em Modo SolrCloud

O Solr é implantado em modo SolrCloud, que requer um ensemble Zookeeper para coordenação distribuída, gerenciamento de configurações de collections, eleição de líder de shard e descoberta de nós. O chart Helm do Solr geralmente inclui uma opção para implantar um Zookeeper interno ou se conectar a um externo. Neste caso, um Zookeeper interno é configurado.

-   **Zookeeper Habilitado (`zookeeper.enabled`):**
    -   **Valor:** `true`
    -   **Descrição Detalhada:** Indica que o chart Helm gerenciará a implantação de um ensemble Zookeeper para uso pelo SolrCloud. Se fosse `false`, o Solr precisaria ser configurado para se conectar a um Zookeeper externo existente.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `zookeeper.enabled`)

-   **Flags JVM para Zookeeper (`zookeeper.jvmFlags`):**
    -   **Valor:** `"-Djute.maxbuffer=5000000"`
    -   **Descrição Detalhada:** Passa flags específicas para a JVM dos pods Zookeeper. `-Djute.maxbuffer=5000000` aumenta o tamanho máximo do buffer para mensagens Jute (o protocolo de serialização do Zookeeper) para aproximadamente 5MB. Isso é importante porque o Solr armazena informações de configuração consideráveis no Zookeeper (ex: `schema.xml`, `solrconfig.xml`, estado do cluster). Se essas configurações forem grandes, o buffer padrão do Jute (geralmente 1MB) pode ser excedido, causando falhas de comunicação entre Solr e Zookeeper. Este mesmo valor é também configurado nos `SOLR_OPTS` (Seção 3.3) para os clientes Solr.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `zookeeper.jvmFlags`)

-   **Imagem do Zookeeper (`zookeeper.image`):**
    -   **Repositório (`zookeeper.image.repository`):** `zookeeper`
    -   **Descrição Detalhada:** Especifica o nome da imagem Zookeeper a ser usada, provavelmente do registro global `registry.rancher.tcu.gov.br/aplicacoes`. A tag da imagem não está explicitamente definida aqui, o que significa que o chart Helm do Zookeeper (que é um sub-chart ou parte do chart Solr) usará sua tag padrão ou uma tag definida globalmente, se aplicável.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `zookeeper.image.repository`)

-   **Serviço do Zookeeper (`zookeeper.service`):**
    -   **Tipo (`zookeeper.service.type`):** `NodePort`
        -   **Descrição Detalhada:** Expõe o serviço Zookeeper usando o tipo `NodePort`. Isso significa que a porta do cliente Zookeeper será exposta em uma porta estática em cada nó do cluster Kubernetes. Embora `NodePort` possa ser usado para acesso externo (com ressalvas), aqui é mais provável que seja para garantir um endpoint acessível consistentemente dentro do cluster, ou para fins de diagnóstico, já que os pods Solr normalmente se conectariam ao serviço Zookeeper usando seu nome de serviço ClusterIP interno (ex: `solr-zookeeper.solr.svc.cluster.local`).
    -   **Porta do Cliente NodePort (`zookeeper.service.nodePorts.client`):** `32181`
        -   **Descrição Detalhada:** Especifica a porta estática (`32181`) que será usada nos nós do cluster para expor a porta do cliente Zookeeper (que é, por padrão, a 2181). Assim, o Zookeeper seria acessível em `<IP-do-No>:32181`.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `zookeeper.service`)

-   **Recursos para Zookeeper (`zookeeper.resources`):**
    -   **Limites (`zookeeper.resources.limits`):** `{}` (Vazio)
        -   **Descrição Detalhada:** Semelhante aos pods Solr, não há limites de CPU ou memória explicitamente definidos para os pods Zookeeper. Isso acarreta os mesmos riscos potenciais de consumo excessivo de recursos se não houver `LimitRange` no namespace.
    -   **Requisições (`zookeeper.resources.requests`):**
        -   CPU: `250m` (0.25 core)
        -   Memória: `1Gi`
        -   **Descrição Detalhada:** Reserva recursos modestos para cada pod Zookeeper. O Zookeeper geralmente não é tão intensivo em recursos quanto o Solr, mas requer memória suficiente para manter os dados de estado em memória e CPU para lidar com as requisições de coordenação.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `zookeeper.resources`)

-   **Persistência do Zookeeper (`zookeeper.persistence`):**
    O Zookeeper também requer armazenamento persistente para seus logs de transação e snapshots de dados.
    -   **Habilitada (`zookeeper.persistence.enabled`):** `true`
    -   **Política de Recuperação (`zookeeper.persistence.reclaimPolicy`):** `Retain`
    -   **Classe de Armazenamento (`zookeeper.persistence.storageClass`):** `sc-solr`
    -   **Modos de Acesso (`zookeeper.persistence.accessModes`):** `[ReadWriteOnce]`
    -   **Tamanho (`zookeeper.persistence.size`):** `100Gi`
    -   **Descrição Detalhada:** Estas configurações são análogas às da persistência do Solr (Seção 3.9). Cada pod Zookeeper (geralmente 3 ou 5 réplicas para um ensemble robusto) terá seu próprio PVC de 100Gi, usando a mesma `StorageClass` `sc-solr` e política de retenção `Retain`.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `zookeeper.persistence`)

-   **Seletor de Nó para Zookeeper (`zookeeper.nodeSelector`):**
    -   **Valor:** `solr: "true"`
    -   **Descrição Detalhada:** Agenda os pods Zookeeper nos mesmos nós dedicados ao Solr (com o rótulo `solr: "true"`). Isso pode ser feito para co-localizar os componentes ou se os nós Solr tiverem características (ex: rede rápida, discos) que também beneficiam o Zookeeper.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `zookeeper.nodeSelector`)

-   **Tolerâncias para Zookeeper (`zookeeper.tolerations`):**
    -   **Valor:** `effect: NoSchedule, key: solr, value: "true", operator: Equal`
    -   **Descrição Detalhada:** Permite que os pods Zookeeper sejam agendados nos nós com o taint `solr=true:NoSchedule`.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `zookeeper.tolerations`)

-   **Rótulos do Pod para Zookeeper (`zookeeper.podLabels`):**
    -   **Valor:** `app: zookeeper`
    -   **Descrição Detalhada:** Adiciona o rótulo `app: zookeeper` aos pods Zookeeper, útil para seleção e para regras de afinidade.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `zookeeper.podLabels`)

-   **Afinidade para Zookeeper (`zookeeper.affinity`):**
    -   **Valor:** `podAntiAffinity` configurado com `requiredDuringSchedulingIgnoredDuringExecution` para `app: zookeeper` e `topologyKey: "kubernetes.io/hostname"`.
    -   **Descrição Detalhada:** Similar à anti-afinidade dos pods Solr (Seção 3.4), esta regra garante que os pods Zookeeper sejam distribuídos em diferentes nós para alta disponibilidade do ensemble Zookeeper. A falha de um nó não deve derrubar múltiplas instâncias do Zookeeper, o que poderia comprometer o quorum e, consequentemente, todo o cluster SolrCloud.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (seção: `zookeeper.affinity`)

-   **Política de Rede para Zookeeper (`zookeeper.networkPolicy.enabled`):**
    -   **Valor:** `false`
    -   **Descrição Detalhada:** Indica que nenhuma `NetworkPolicy` específica para o Zookeeper é criada por este chart. O tráfego de e para os pods Zookeeper será governado por políticas de rede globais ou do namespace, se existirem. Em um ambiente seguro, seria ideal ter NetworkPolicies que restrinjam o acesso aos pods Zookeeper apenas aos pods Solr e outras entidades autorizadas.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `zookeeper.networkPolicy.enabled`)

### 3.12. Política de Rede (NetworkPolicy) para SOLR

NetworkPolicies no Kubernetes controlam o fluxo de tráfego de rede entre pods.

-   **Política de Rede Habilitada (`networkPolicy.enabled`):**
    -   **Valor:** `false`
    -   **Descrição Detalhada:** Indica que o chart Helm do Solr não criará um recurso `NetworkPolicy` específico para os pods Solr. Isso significa que, por padrão, os pods Solr podem aceitar tráfego de qualquer origem dentro do cluster (ou de fora, se expostos por um Service ou Ingress) e podem iniciar conexões para qualquer destino, sujeito a quaisquer políticas de rede mais amplas que possam estar em vigor no namespace ou no cluster.
    -   **Considerações de Segurança:** Para uma implantação segura, é altamente recomendável habilitar e configurar NetworkPolicies. Uma política bem definida para o Solr restringiria o tráfego de entrada para apenas as fontes necessárias (ex: pods da aplicação que consultam o Solr, o Ingress Controller, outros nós Solr para comunicação interna do cluster) e limitaria o tráfego de saída (ex: para o Zookeeper, para o sistema de backup, para o Vault, etc.). A ausência de uma NetworkPolicy específica aumenta a superfície de ataque.
    -   **Referência GitOps:** `apps/solr/solr/values.yaml` (linha: `networkPolicy.enabled`)

---


## 4. Gerenciamento da Aplicação Solr via Fleet (`apps/solr/solr/fleet.yaml`)

O Fleet é uma ferramenta do Rancher que permite gerenciar implantações de aplicações e configurações em múltiplos clusters Kubernetes usando uma abordagem GitOps. O arquivo `fleet.yaml` localizado em `apps/solr/solr/fleet.yaml` define como a aplicação Solr principal (configurada pelo chart Helm e `values.yaml` descritos anteriormente) é gerenciada e implantada pelo Fleet.

**Conteúdo e Análise do `apps/solr/solr/fleet.yaml`:**

```yaml
defaultNamespace: solr
name: solr
namespaceLabels:
  team: seint
helm:
  releaseName: solr
  chart: oci://registry.rancher.tcu.gov.br/aplicacoes/solr
  #version: 8.10.0
  #version: 9.2.1
  #version: 9.4.1
  version: 9.5.1
  valuesFiles:
    - values.yaml

kustomize:
  dir: ./security
```

-   **`defaultNamespace: solr`**
    -   **Descrição Detalhada:** Especifica que, por padrão, os recursos definidos por este bundle do Fleet (neste caso, a aplicação Solr) serão implantados no namespace `solr` dentro do cluster Kubernetes de destino. Se o namespace não existir, o Fleet geralmente o criará.

-   **`name: solr`**
    -   **Descrição Detalhada:** Define o nome deste bundle do Fleet como `solr`. Este nome é usado para identificar e gerenciar este conjunto de configurações dentro da interface do Fleet no Rancher.

-   **`namespaceLabels:`**
    -   **`team: seint`**
    -   **Descrição Detalhada:** Aplica o rótulo `team: seint` ao namespace `solr` (se o Fleet o criar ou gerenciá-lo). Rótulos em namespaces são úteis para organização, aplicação de políticas (como NetworkPolicies ou cotas de recursos por equipe) e para filtragem.

-   **`helm:`**
    Esta seção configura como o Fleet usará o Helm para implantar a aplicação Solr.
    -   **`releaseName: solr`**
        -   **Descrição Detalhada:** Define o nome da release Helm que será criada ou atualizada. O nome da release Helm é usado para rastrear a implantação específica gerenciada pelo Helm.
    -   **`chart: oci://registry.rancher.tcu.gov.br/aplicacoes/solr`**
        -   **Descrição Detalhada:** Especifica a localização do chart Helm a ser usado. Neste caso, o chart é armazenado como um artefato OCI (Open Container Initiative) no registro privado do TCU: `registry.rancher.tcu.gov.br/aplicacoes/solr`. Armazenar charts Helm como imagens OCI é uma prática moderna que simplifica a distribuição e o versionamento de charts.
    -   **`version: 9.5.1`**
        -   **Descrição Detalhada:** Define a versão específica do chart Helm a ser usada (`9.5.1`). É crucial fixar a versão do chart para garantir implantações consistentes e evitar atualizações inesperadas se uma nova versão do chart for publicada no registro com o mesmo nome genérico (sem tag de versão) ou se o Fleet fosse configurado para usar a versão mais recente.
    -   **`valuesFiles:`**
        -   **`- values.yaml`**
        -   **Descrição Detalhada:** Especifica uma lista de arquivos de valores que serão usados para configurar o chart Helm. Aqui, ele aponta para o arquivo `values.yaml` localizado no mesmo diretório (`apps/solr/solr/values.yaml`), que contém todas as configurações detalhadas nas seções 3.1 a 3.12 deste documento.

-   **`kustomize:`**
    Esta seção configura como o Fleet usará o Kustomize para aplicar customizações sobre os manifestos gerados pelo Helm ou para adicionar manifestos adicionais.
    -   **`dir: ./security`**
        -   **Descrição Detalhada:** Instrui o Fleet a aplicar as customizações definidas no diretório `./security` (relativo à localização deste `fleet.yaml`, ou seja, `apps/solr/solr/security/`). O Kustomize neste diretório é usado para gerenciar o Segredo Kubernetes que contém a política de segurança Java (`security.policy`), conforme detalhado na Seção 5 e 6.

**Fluxo de Implantação via Fleet:**

1.  O Fleet, configurado no Rancher, monitora o repositório Git onde este arquivo `fleet.yaml` (e os demais arquivos de configuração) reside.
2.  Quando uma alteração é detectada neste arquivo ou nos arquivos referenciados (como `values.yaml` ou os arquivos no diretório `security/`), o Fleet inicia o processo de implantação ou atualização.
3.  O Fleet primeiro processa a seção `helm`:
    -   Baixa o chart Helm `solr` versão `9.5.1` do registro OCI.
    -   Renderiza os manifestos Kubernetes usando este chart e o arquivo `values.yaml` fornecido.
4.  Em seguida, o Fleet processa a seção `kustomize`:
    -   Aplica as transformações ou gera os manifestos adicionais definidos no diretório `apps/solr/solr/security/` (que, neste caso, cria o Segredo `solr-policy`).
5.  O Fleet então aplica o conjunto final de manifestos Kubernetes (resultantes do Helm e do Kustomize) ao cluster de destino, no namespace `solr`.

**Vantagens da Abordagem com Fleet:**
-   **GitOps:** Mantém o Git como a fonte da verdade, promovendo rastreabilidade e auditoria.
-   **Automação:** Implantações e atualizações são automatizadas.
-   **Consistência:** Garante que o estado do cluster reflita o estado definido no Git.
-   **Gerenciamento Centralizado:** Permite gerenciar múltiplas aplicações e clusters a partir de uma única interface (Rancher) e repositório Git.

---


## 5. Gerenciamento da Política de Segurança Java via Kustomize (`apps/solr/solr/security/kustomization.yaml`)

Kustomize é uma ferramenta standalone para customizar manifestos Kubernetes. Ela permite sobrepor ou adicionar configurações a um conjunto base de manifestos sem modificar os originais (template-free way). No contexto da implantação do Solr, o Kustomize é utilizado para gerenciar a criação do Segredo Kubernetes que contém a política de segurança Java (`security.policy`). Este Segredo é então montado nos pods Solr, conforme descrito na Seção 3.6.

O arquivo `kustomization.yaml` localizado em `apps/solr/solr/security/kustomization.yaml` define como este Segredo é gerado.

**Conteúdo e Análise do `apps/solr/solr/security/kustomization.yaml`:**

```yaml
secretGenerator:
  - name: solr-policy
    files:
      - security.policy
    type: Opaque

generatorOptions:
  disableNameSuffixHash: true # fixar nome da secret
```

-   **`secretGenerator:`**
    Esta seção instrui o Kustomize a gerar um recurso `Secret` do Kubernetes.
    -   **`- name: solr-policy`**
        -   **Descrição Detalhada:** Define o nome do Segredo Kubernetes a ser gerado como `solr-policy`. Este é o mesmo nome de segredo referenciado na seção `extraVolumes` do `values.yaml` do Solr (ver Seção 3.6) para montar a política de segurança.
    -   **`files:`**
        -   **`- security.policy`**
        -   **Descrição Detalhada:** Especifica os arquivos cujos conteúdos serão incluídos no Segredo. Aqui, ele aponta para o arquivo `security.policy` localizado no mesmo diretório (`apps/solr/solr/security/security.policy`). O conteúdo deste arquivo se tornará os dados da chave `security.policy` dentro do Segredo `solr-policy`.
    -   **`type: Opaque`**
        -   **Descrição Detalhada:** Define o tipo do Segredo como `Opaque`. Este é o tipo padrão para segredos que contêm dados arbitrários definidos pelo usuário, como arquivos de configuração ou credenciais.

-   **`generatorOptions:`**
    Esta seção fornece opções que controlam como os geradores (como `secretGenerator`) se comportam.
    -   **`disableNameSuffixHash: true`**
        -   **Descrição Detalhada:** Por padrão, o Kustomize anexa um sufixo de hash ao nome dos recursos que ele gera (como ConfigMaps e Secrets) para garantir que as atualizações nos dados resultem em um novo nome de recurso, forçando um rolling update dos pods que os utilizam. Ao definir `disableNameSuffixHash: true`, este comportamento é desabilitado para este gerador específico. Isso significa que o nome do Segredo gerado será exatamente `solr-policy`, sem qualquer sufixo de hash. Isso é crucial aqui porque o `values.yaml` do Solr referencia o segredo pelo nome fixo `solr-policy`. Se o nome do segredo mudasse a cada alteração na política, a referência no `values.yaml` (ou a montagem do volume no pod) precisaria ser atualizada, ou o pod não conseguiria montar a nova versão do segredo.
        -   **Implicação:** Com o nome do segredo fixo, quando o conteúdo do arquivo `security.policy` muda e o Kustomize regenera o segredo, os pods Solr existentes precisarão ser reiniciados (geralmente através de um rolling update acionado pelo Fleet ou manualmente) para carregar a nova versão da política de segurança do Segredo atualizado.

**Integração com o Fluxo do Fleet:**

Conforme visto na Seção 4, o arquivo `fleet.yaml` principal para o Solr (`apps/solr/solr/fleet.yaml`) inclui uma seção `kustomize`:

```yaml
kustomize:
  dir: ./security
```

Isso significa que o Fleet, após processar o chart Helm, aplicará as customizações definidas pelo `kustomization.yaml` no diretório `security/`. O Kustomize lerá o `apps/solr/solr/security/kustomization.yaml`, gerará o Segredo `solr-policy` com base no conteúdo do arquivo `apps/solr/solr/security/security.policy`, e este Segredo será então aplicado ao cluster Kubernetes.

**Procedimento de Manutenção (Atualização da Política de Segurança Java):**

1.  **Editar `security.policy`:** O administrador modifica o arquivo `apps/solr/solr/security/security.policy` no repositório Git para adicionar, remover ou alterar permissões Java.
2.  **Commit e Push:** As alterações são versionadas no Git.
3.  **Implantação via GitOps:**
    -   O Fleet detecta a mudança no arquivo `security.policy` (ou no `kustomization.yaml`, se ele fosse alterado).
    -   O Kustomize, como parte do processo do Fleet, regenera o Segredo `solr-policy` com o novo conteúdo.
    -   O Fleet aplica o Segredo `solr-policy` atualizado ao cluster.
4.  **Reinício dos Pods Solr:** Como o nome do Segredo é fixo e os volumes de segredo montados nos pods são atualizados dinamicamente pelo Kubernetes, os pods Solr existentes verão a mudança no arquivo `security.policy` montado. No entanto, para que a JVM do Solr carregue e aplique a nova política de segurança, um reinício dos pods Solr é geralmente necessário. O Fleet pode ser configurado para acionar um rolling update dos pods Solr se o Segredo do qual eles dependem for alterado, ou isso pode precisar ser feito manualmente.

Este mecanismo garante que a política de segurança Java, um componente crítico da segurança do Solr, seja gerenciada como código, versionada e implantada de forma consistente através do pipeline GitOps.

---


## 6. Análise Detalhada da Política de Segurança Java (`apps/solr/solr/security/security.policy`)

O arquivo `security.policy` localizado em `apps/solr/solr/security/security.policy` é um componente crucial para a segurança da JVM (Java Virtual Machine) que executa o Solr. Quando o Java Security Manager está habilitado (o que é uma prática de segurança recomendada, especialmente para mitigar o impacto de vulnerabilidades de execução remota de código ou acesso não autorizado a recursos do sistema), este arquivo define de forma granular quais permissões são concedidas ao código em execução. Isso inclui o próprio Solr, suas bibliotecas principais (como Lucene), e quaisquer plugins ou drivers de terceiros que ele carregue.

O objetivo principal de uma política de segurança Java é aplicar o princípio do menor privilégio: o código só deve ter as permissões estritamente necessárias para seu funcionamento. Isso limita o dano potencial caso uma vulnerabilidade seja explorada.

**Análise do Conteúdo do `apps/solr/solr/security/security.policy` Fornecido:**

O arquivo `security.policy` fornecido no GitOps é uma política de segurança Java padrão, frequentemente encontrada em distribuições Solr, e é bastante permissiva em muitos aspectos, especialmente porque parece ser a política padrão do Solr que inclui permissões para testes e diversos componentes.

**Estrutura Geral:**
O arquivo é composto por blocos `grant { ... };`. Cada bloco concede um conjunto de permissões. As permissões podem ser condicionadas a uma `codeBase` (URL de onde o código é carregado) ou, como na maioria dos casos neste arquivo, serem globais (aplicadas a todo o código em execução, a menos que uma `codeBase` mais específica também se aplique).

**Principais Seções e Permissões Concedidas:**

1.  **Permissões para Testes e Ambiente de Build:**
    *   Muitas das permissões iniciais, como acesso a `${common.dir}`, `${junit4.childvm.cwd}`, `${user.home}/.ivy2/cache/`, são tipicamente relacionadas ao ambiente de desenvolvimento e teste do Solr. Em um ambiente de produção estritamente configurado, estas poderiam ser removidas ou severamente restringidas se não forem necessárias para a operação normal do Solr.
    *   Exemplos: `permission java.io.FilePermission "${common.dir}${/}-", "read";`, `permission java.io.FilePermission "${junit4.childvm.cwd}${/}temp${/}-", "read,write,delete";`.

2.  **Permissões Fundamentais da JVM e Sistema:**
    *   `permission java.io.FilePermission "${java.home}${/}-", "read";`: Permite ler arquivos da instalação Java (JRE/JDK).
    *   `permission java.io.FilePermission "${java.io.tmpdir}", "read,write";` e `permission java.io.FilePermission "${java.io.tmpdir}${/}-", "read,write,delete";`: Concede acesso de leitura, escrita e exclusão ao diretório temporário do sistema e seus subdiretórios. Essencial para muitas operações.
    *   `permission java.util.PropertyPermission "*", "read,write";`: **Esta é uma permissão muito ampla.** Concede ao código a capacidade de ler e escrever *qualquer* propriedade de sistema Java. Em um ambiente de produção seguro, seria preferível listar explicitamente apenas as propriedades que o Solr precisa ler ou escrever.
    *   `permission java.lang.reflect.ReflectPermission "suppressAccessChecks";` e `permission java.lang.RuntimePermission "accessDeclaredMembers";`: Necessárias para frameworks que usam reflexão extensivamente, como o próprio Solr e muitas bibliotecas Java.
    *   Diversas `java.lang.RuntimePermission` como `setIO`, `setDefaultUncaughtExceptionHandler`, `modifyThreadGroup`, `getStackTrace`, `createClassLoader`, `shutdownHooks`, `getenv.*`, `getClassLoader`, `setContextClassLoader`, `defineClass`. Estas são permissões poderosas, muitas vezes necessárias para o funcionamento de servidores de aplicação complexos e seus componentes (logging, carregamento de plugins, instrumentação, etc.).

3.  **Permissões de Rede:**
    *   `permission java.net.SocketPermission "localhost:1024-", "accept,listen,connect,resolve";` (e variações para `127.0.0.1`, `[::1]`): Permite que o Solr escute, aceite conexões e se conecte a portas `>=1024` em `localhost`. Isso é fundamental para a comunicação interna do Solr e para expor seus serviços.
    *   `permission java.net.SocketPermission "${solr.internal.network.permission}", "accept,listen,connect,resolve";`: Se a propriedade de sistema `solr.internal.network.permission` for definida (ex: para `*`), isso pode conceder permissões de rede muito amplas. É importante verificar como essa propriedade é definida no ambiente de execução.

4.  **Permissões Específicas do Solr (baseadas em propriedades de sistema):**
    O segundo bloco `grant` concede permissões de arquivo com base em propriedades de sistema que são tipicamente definidas pelo script de inicialização do Solr (`bin/solr`):
    *   `permission java.io.FilePermission "${solr.install.dir}${/}-", "read,write,delete,readlink";`: Acesso ao diretório de instalação do Solr.
    *   `permission java.io.FilePermission "${jetty.home}${/}-", "read,write,delete,readlink";`: Acesso ao diretório home do Jetty (o servlet container embutido).
    *   `permission java.io.FilePermission "${solr.solr.home}${/}-", "read,write,delete,readlink";`: Acesso ao diretório SOLR_HOME, onde as configurações dos cores residem.
    *   `permission java.io.FilePermission "${solr.data.home}${/}-", "read,write,delete,readlink";`: Acesso ao diretório de dados onde os índices são armazenados.
    *   `permission java.io.FilePermission "${solr.log.dir}${/}-", "read,write,delete,readlink";`: Acesso ao diretório de logs.
    *   `permission java.io.FilePermission "${solr.allowPaths}${/}-", "read,write,delete,readlink";`: Se `solr.allowPaths` for definido (ex: como `*` nos `SOLR_OPTS`), esta permissão concede acesso aos caminhos especificados. A configuração `-Dsolr.allowPaths=*` nos `SOLR_OPTS` (Seção 3.3) torna esta permissão muito ampla.
    *   Acesso a arquivos de keystore/truststore (`${solr.jetty.keystore}`, `${javax.net.ssl.trustStore}`, etc.).

5.  **Permissões para Componentes e Integrações:**
    *   **JMX (Java Management Extensions):** Permissões `javax.management.*` para permitir o monitoramento e gerenciamento do Solr via JMX (ex: `getAttribute`, `registerMBean`).
    *   **Logging:** `permission java.util.logging.LoggingPermission "control";`.
    *   **Hadoop/AWS S3:** Permissões relacionadas à segurança do Hadoop (`javax.security.auth.AuthPermission`, `javax.security.auth.PrivateCredentialPermission`) e acesso a arquivos de credenciais da AWS (`${aws.sharedCredentialsFile}`). Isso sugere que o Solr pode estar configurado ou ter a capacidade de interagir com HDFS ou S3 (ex: para backups ou índices em S3), embora a configuração principal no `values.yaml` não detalhe explicitamente essa integração para dados primários.
    *   **JAAS (Java Authentication and Authorization Service):** Permissões para `javax.security.auth.kerberos.*` indicam suporte para autenticação Kerberos.
    *   **DIH (Data Import Handler):** `permission java.sql.SQLPermission "deregisterDriver";`.

6.  **Permissões Específicas para API do TCU:**
    O terceiro bloco `grant` é uma customização específica para o ambiente do TCU:
    ```java
    grant {
        permission java.net.URLPermission "https://contas.tcu.gov.br/ords/apex_cedoc_vce_p/termos/sinonimo/*", "GET";
        permission java.net.URLPermission "https://contas.tcu.gov.br/ords/apex_cedoc_vce_p/termos/cod/*", "GET";
    };
    ```
    *   **Descrição Detalhada:** Concede explicitamente ao Solr a permissão para fazer requisições HTTP GET para duas URLs específicas no domínio `contas.tcu.gov.br`. Isso é provavelmente usado para buscar dinamicamente listas de sinônimos ou outros termos/códigos de uma API interna do TCU, que são então usados para enriquecer a indexação ou a busca no Solr. Esta é uma boa prática de especificar permissões de rede para URLs exatas em vez de usar wildcards amplos.

**Considerações de Segurança e Recomendações:**

-   **Princípio do Menor Privilégio:** A política fornecida é bastante ampla em vários pontos (ex: `PropertyPermission "*", "read,write"`, `FilePermission` baseada em `solr.allowPaths=*`). Para um ambiente de produção com segurança reforçada, esta política deveria ser revisada e restringida ao mínimo necessário. Cada permissão `*` (wildcard) deve ser avaliada.
-   **Remoção de Permissões de Teste:** Permissões claramente relacionadas apenas a ambientes de teste/build poderiam ser removidas.
-   **Habilitação do Security Manager:** Esta política só tem efeito se o Java Security Manager for efetivamente habilitado ao iniciar a JVM do Solr. Isso é geralmente feito passando o argumento `-Djava.security.manager -Djava.security.policy==/path/to/security.policy` para a JVM. É crucial verificar se o Security Manager está de fato ativo na implantação do TCU.
-   **Manutenção Contínua:** À medida que plugins são adicionados ou a configuração do Solr muda, a política de segurança pode precisar ser atualizada. Qualquer nova permissão deve ser cuidadosamente avaliada.

**Relação com `-Dsolr.allowPaths=*`:**
A configuração `-Dsolr.allowPaths=*` nos `SOLR_OPTS` (Seção 3.3) instrui o Solr a permitir acesso a qualquer caminho para certas funcionalidades. A `java.io.FilePermission "${solr.allowPaths}${/}-", "read,write,delete,readlink";` na política de segurança então concede as permissões de sistema de arquivos correspondentes. Se `solr.allowPaths` fosse mais restritivo (ex: listando apenas diretórios de backup específicos), a superfície de ataque seria reduzida.

**Conclusão sobre a Política de Segurança:**
A política de segurança Java em vigor é um misto de permissões padrão do Solr (que são relativamente abertas para cobrir muitos casos de uso e testes) e customizações específicas do TCU (como o acesso à API de termos). Para fortalecer a segurança, uma análise detalhada para restringir permissões desnecessárias, especialmente wildcards, é recomendada, garantindo ao mesmo tempo que todas as funcionalidades necessárias do Solr e dos plugins customizados continuem operando corretamente.

---


## 7. Procedimentos Detalhados de Manutenção do Ambiente Solr

A manutenção proativa e reativa do ambiente Solr é crucial para garantir sua disponibilidade contínua, desempenho ótimo e integridade dos dados. Com a abordagem GitOps, muitos aspectos da manutenção são definidos como código e automatizados. Esta seção detalha os principais procedimentos de manutenção com base na configuração do repositório GitOps fornecido.

### 7.1. Gerenciamento de Backups Automatizados (`apps/solr/solr-backups/`)

A capacidade de realizar backups confiáveis e regulares das collections Solr é uma das pedras angulares da estratégia de recuperação de desastres e da proteção de dados. O diretório `apps/solr/solr-backups/` no GitOps contém as configurações para automatizar este processo utilizando `CronJob` do Kubernetes, que executam comandos para acionar a API de Backup do Solr.

#### 7.1.1. Orquestração dos Backups via Fleet

O arquivo `apps/solr/solr-backups/fleet.yaml` é responsável por agrupar e gerenciar a implantação dos diversos CronJobs de backup no cluster Kubernetes através do Fleet.

**Conteúdo e Análise do `apps/solr/solr-backups/fleet.yaml`:**

```yaml
defaultNamespace: solr
name: solr-backups

#######
#
# EXEMPLO DE CRONJOB
# ... (comentários explicativos sobre como configurar um CronJob de backup)
# ...
######
#
# kind: CronJob
# ... (exemplo de manifesto CronJob comentado)
# ...
```

-   **`defaultNamespace: solr`**
    -   **Descrição Detalhada:** Garante que todos os CronJobs de backup definidos neste bundle do Fleet sejam implantados no namespace `solr`. Isso mantém os recursos de backup organizados junto com a aplicação Solr principal.

-   **`name: solr-backups`**
    -   **Descrição Detalhada:** Define o nome deste bundle do Fleet como `solr-backups`. Este nome é usado para identificar este conjunto de configurações de backup na interface do Fleet no Rancher.

-   **Comentários e Exemplo de CronJob:**
    -   **Descrição Detalhada:** O `fleet.yaml` em `solr-backups` é um pouco diferente dos outros `fleet.yaml` analisados. Ele não parece definir um `targetCustomizations` ou uma lista explícita de arquivos a serem aplicados. Em vez disso, ele contém extensos comentários que servem como um **template e documentação** sobre como criar e configurar os arquivos YAML individuais dos CronJobs de backup que residem no mesmo diretório (ex: `cron-backup-acordao.yaml`).
    -   Isso sugere que cada arquivo `cron-backup-*.yaml` no diretório `apps/solr/solr-backups/` é provavelmente gerenciado como um recurso individual pelo Fleet (talvez cada um sendo um `GitRepo` separado ou parte de um bundle maior que apenas aplica todos os YAMLs de um diretório), ou que o Fleet está configurado para aplicar todos os manifestos YAML válidos encontrados neste caminho do repositório.
    -   Os comentários detalham quais campos devem ser alterados (`metadata.name`, `containers.name`, `command`, `schedule`, `env.COLLECTION`, `env.NAME`) e quais devem ser mantidos (`env.PASS`, `env.ASYNC`, `env.DOMINIO`, `env.LOCATION`).

**Funcionamento da Orquestração:**
O Fleet monitora o diretório `apps/solr/solr-backups/`. Qualquer adição, modificação ou remoção de arquivos `cron-backup-*.yaml` neste diretório será detectada pelo Fleet e as alterações correspondentes (criação, atualização ou exclusão de CronJobs) serão aplicadas ao cluster Kubernetes. Isso garante que a estratégia de backup esteja sempre sincronizada com o que está definido no Git.

#### 7.1.2. Estrutura e Funcionamento Detalhado dos CronJobs de Backup

Cada arquivo `cron-backup-*.yaml` (ex: `cron-backup-acordao.yaml`, `cron-backup-dou.yaml`, etc.) no diretório `apps/solr/solr-backups/` define um recurso `CronJob` do Kubernetes. Um `CronJob` cria Jobs em uma programação recorrente definida no formato cron.

**Análise de um Arquivo de Exemplo (`cron-backup-acordao.yaml`):**

```yaml
kind: CronJob
apiVersion: batch/v1
metadata:
  name: solr-bkp-acordao-completo
spec:
  concurrencyPolicy: Forbid
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          nodeSelector:
            solr: "true"
          tolerations:
          - effect: NoSchedule
            key: solr
            value: "true"
            operator: Equal
          containers:
            - command:
                - /bin/bash
                - '-ec'
                - >
                  curl -v -u admin:${PASS} --location "http://${DOMINIO}/solr/admin/collections?action=BACKUP&collection=${COLLECTION}&location=${LOCATION}&name=${NAME}&async=$(ASYNC)"
              env:
                - name: DOMINIO
                  value: 'solr.solr.svc.cluster.local:8983'
                - name: COLLECTION
                  value: 'acordao-completo'
                - name: LOCATION
                  value: '/bitnami/backups/'
                - name: NAME
                  value: acordao-completo-bkp
                - name: PASS
                  valueFrom:
                    secretKeyRef:
                      key: solr-password
                      name: solr-usr-secret
                - name: ASYNC
                  value: "$(date +%Y%m%d%H%M%S)"
              image: bitnami/os-shell # Ou uma imagem Solr específica
              imagePullPolicy: Always
              name: solr-bkp-acordao-completo
              terminationMessagePath: /dev/termination-log
              terminationMessagePolicy: File
          dnsPolicy: ClusterFirst
          restartPolicy: Never
          schedulerName: default-scheduler
          terminationGracePeriodSeconds: 30
  timeZone: "America/Sao_Paulo"
  schedule: '0 11 * * *' # Exemplo: Diariamente às 11:00 AM (horário de São Paulo)
  successfulJobsHistoryLimit: 3
  suspend: false
```

-   **`kind: CronJob`, `apiVersion: batch/v1`**: Define o tipo de recurso Kubernetes.
-   **`metadata.name`**: Nome único para o CronJob (ex: `solr-bkp-acordao-completo`).
-   **`spec.schedule`**: Define a frequência de execução no formato cron (ex: `'0 11 * * *'` para diariamente às 11:00). O `timeZone: "America/Sao_Paulo"` garante que a agenda seja interpretada corretamente.
-   **`spec.concurrencyPolicy: Forbid`**: Impede que um novo Job seja iniciado se o Job anterior ainda estiver em execução.
-   **`spec.failedJobsHistoryLimit` e `spec.successfulJobsHistoryLimit`**: Controlam quantos Jobs concluídos (com falha ou sucesso) são mantidos no histórico do Kubernetes para fins de auditoria e depuração.
-   **`spec.jobTemplate`**: Define o template para os Jobs que o CronJob criará.
    -   **`spec.template.spec.nodeSelector` e `spec.template.spec.tolerations`**: Garantem que os pods dos Jobs de backup sejam agendados nos mesmos nós dedicados ao Solr (com rótulo `solr: "true"` e tolerância ao taint correspondente). Isso pode ser útil para acesso eficiente à rede do Solr ou se os nós tiverem montagens de volume específicas.
    -   **`spec.template.spec.restartPolicy: Never`**: Se o contêiner do Job falhar, o Pod não será reiniciado; o Job em si será marcado como falho.
    -   **`spec.template.spec.containers`**: Define o contêiner que executará a lógica de backup.
        -   **`name`**: Nome do contêiner (ex: `solr-bkp-acordao-completo`).
        -   **`image`**: Imagem Docker a ser usada. Pode ser uma imagem leve com `curl` e `bash` (como `bitnami/os-shell`) ou uma imagem Solr completa. Usar `bitnami/os-shell` é eficiente se apenas `curl` for necessário.
        -   **`command`**: O comando principal que executa o backup. Utiliza `curl` para chamar a API de Backup do Solr (`/solr/admin/collections?action=BACKUP`).
            -   `-u admin:${PASS}`: Autenticação básica. A senha (`${PASS}`) é injetada via variável de ambiente.
            -   `collection=${COLLECTION}`: Especifica a collection Solr a ser backupeada.
            -   `location=${LOCATION}`: O diretório *dentro dos pods Solr* onde o backup será armazenado. Este caminho (`/bitnami/backups/`) é o ponto de montagem do volume NFS `backup-nfs` (configurado no `values.yaml` do Solr, Seção 3.6), garantindo que os dados do backup sejam escritos no armazenamento NFS compartilhado.
            -   `name=${NAME}`: O nome do snapshot de backup que será criado no Solr.
            -   `async=$(ASYNC)`: Permite que a operação de backup seja assíncrona. O valor de `ASYNC` é um timestamp gerado dinamicamente (`$(date +%Y%m%d%H%M%S)`) para fornecer um ID de requisição único.
        -   **`env` (Variáveis de Ambiente):** Parametrizam o comando `curl`.
            -   `DOMINIO`: `solr.solr.svc.cluster.local:8983` (URL do serviço ClusterIP interno do Solr).
            -   `COLLECTION`: Nome da collection específica para este Job (ex: `acordao-completo`).
            -   `LOCATION`: `/bitnami/backups/` (Caminho de destino no NFS, acessível pelos pods Solr).
            -   `NAME`: Nome do backup (ex: `acordao-completo-bkp`).
            -   `PASS`: Obtém a senha do Solr do Segredo Kubernetes `solr-usr-secret` (chave `solr-password`), que é gerenciado pelo External Secrets (Seção 7.2).
            -   `ASYNC`: ID da requisição assíncrona.

**Arquivos de CronJob Existentes:**
O diretório `apps/solr/solr-backups/` contém múltiplos desses arquivos, um para cada collection importante, como:
-   `cron-backup-acordao.yaml`
-   `cron-backup-boletim-jurisprudencia.yaml`
-   `cron-backup-boletim-pessoal.yaml`
-   `cron-backup-dou.yaml`
-   `cron-backup-informativo-lc.yaml`
-   `cron-backup-jurisprudencia-selecionada.yaml`
-   `cron-backup-ministros.yaml`
-   `cron-backup-norma.yaml`
-   `cron-backup-proc_externo.yaml`
-   `cron-backup-proc_interno.yaml`
-   `cron-backup-sumula.yaml`

Cada um segue o padrão acima, variando principalmente `metadata.name`, `env.COLLECTION`, `env.NAME`, e, em alguns casos, a `schedule` ou a tag da imagem utilizada.

#### 7.1.3. Procedimento Prático para Criação ou Modificação de um Job de Backup

1.  **Definir Requisitos:** Identificar a collection a ser backupeada, a frequência desejada (`schedule`), e o nome do backup.
2.  **Criar Novo Arquivo YAML (ou Copiar Existente):** No diretório `apps/solr/solr-backups/` do repositório Git, criar um novo arquivo (ex: `cron-backup-nova-collection.yaml`) ou copiar um existente e renomeá-lo.
3.  **Modificar Parâmetros:** Editar o arquivo YAML, ajustando os seguintes campos conforme as instruções nos comentários do `fleet.yaml` e o exemplo acima:
    -   `metadata.name` (para o CronJob)
    -   `spec.jobTemplate.spec.template.spec.containers[0].name` (para o contêiner)
    -   `spec.schedule`
    -   `spec.jobTemplate.spec.template.spec.containers[0].env`:
        -   `COLLECTION`
        -   `NAME`
    -   Verificar se a `image` e `imagePullPolicy` estão adequadas.
4.  **Commit e Push:** Salvar o arquivo e enviar as alterações para o repositório Git.
5.  **Verificação:** Após o Fleet aplicar a mudança, verificar no Rancher ou via `kubectl get cronjob -n solr` se o novo CronJob foi criado ou o existente foi atualizado. Monitorar a próxima execução agendada para garantir que o Job de backup seja criado e executado com sucesso.

**Considerações sobre Restauração e Gerenciamento de Espaço:**
-   **Restauração:** Os CronJobs apenas executam backups. O procedimento de restauração de um backup Solr envolve o uso da API de Restauração do Solr, especificando o nome do backup e o local. Este é geralmente um procedimento manual ou que requer um script separado, não coberto por estes CronJobs.
-   **Gerenciamento de Espaço:** Os backups são armazenados no volume NFS `/nas-solr-bkp/producao/`. É crucial ter uma estratégia para gerenciar o espaço de armazenamento, como:
    -   Definir políticas de retenção (quantos backups manter).
    -   Implementar scripts ou processos para excluir backups antigos.
    -   Considerar mover backups mais antigos para um armazenamento de arquivamento de custo mais baixo.
    O Solr não gerencia automaticamente a rotação de backups no local de destino; isso precisa ser tratado externamente.

---


### 7.2. Gerenciamento Centralizado e Seguro de Credenciais com Vault e External Secrets (`apps/solr/solr-secret/`)

O gerenciamento seguro de credenciais, como a senha do usuário administrador do Solr, é uma prática de segurança fundamental. Armazenar senhas diretamente em arquivos de configuração no Git representaria uma vulnerabilidade grave. Para mitigar esse risco, o ambiente Solr no TCU utiliza uma abordagem robusta que integra o HashiCorp Vault (um cofre de segredos centralizado) com o External Secrets Operator (ESO) no Kubernetes. O ESO sincroniza os segredos armazenados no Vault com Segredos Kubernetes nativos, que podem então ser consumidos de forma segura pelas aplicações, como o Solr.

O diretório `apps/solr/solr-secret/` no GitOps contém os manifestos que configuram esta integração.

#### 7.2.1. Orquestração da Configuração de Segredos via Fleet

O arquivo `apps/solr/solr-secret/fleet.yaml` gerencia a implantação dos recursos Kubernetes necessários para a integração com o Vault via Fleet.

**Conteúdo e Análise do `apps/solr/solr-secret/fleet.yaml`:**

```yaml
defaultNamespace: solr
namespaceLabels:
  team: seint
labels:
  team: seint
name: solr-secret
```

-   **`defaultNamespace: solr`**
    -   **Descrição Detalhada:** Especifica que os recursos `SecretStore` e `ExternalSecret`, definidos neste bundle do Fleet, serão criados no namespace `solr`. Isso os co-localiza com a aplicação Solr que consumirá os segredos.

-   **`namespaceLabels:`**
    -   **`team: seint`**
    -   **Descrição Detalhada:** Aplica o rótulo `team: seint` ao namespace `solr`, auxiliando na organização e governança dos recursos.

-   **`labels:`**
    -   **`team: seint`**
    -   **Descrição Detalhada:** Aplica o rótulo `team: seint` ao próprio bundle do Fleet, facilitando sua identificação e gerenciamento dentro do Rancher.

-   **`name: solr-secret`**
    -   **Descrição Detalhada:** Define o nome deste bundle do Fleet como `solr-secret`, indicando claramente seu propósito de gerenciar a configuração de segredos para o Solr.

**Funcionamento da Orquestração:**
O Fleet monitora o diretório `apps/solr/solr-secret/`. Quando este `fleet.yaml` é processado, o Fleet aplica os manifestos `solr-secret-store.yaml` e `solr-secret.yaml` (que devem estar no mesmo diretório ou serem referenciados de outra forma, embora este `fleet.yaml` não os liste explicitamente, é o comportamento padrão do Fleet aplicar todos os YAMLs de um bundle) ao cluster Kubernetes. Isso cria os recursos `SecretStore` e `ExternalSecret` necessários.

#### 7.2.2. Configuração do `SecretStore`: A Ponte para o Vault

O arquivo `apps/solr/solr-secret/solr-secret-store.yaml` define um recurso `SecretStore`. Este é um Custom Resource Definition (CRD) fornecido pelo External Secrets Operator que especifica *como* se conectar a um provedor de segredos externo – neste caso, o HashiCorp Vault.

**Conteúdo e Análise do `apps/solr/solr-secret/solr-secret-store.yaml`:**

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: solr-secret-store
  namespace: solr
spec:
  provider:
    vault:
      auth:
        kubernetes:
          mountPath: kubernetes
          role: solr
          serviceAccountRef:
            name: default
      path: rancher
      server: http://vault.vault:8200
      version: v2
```

-   **`apiVersion: external-secrets.io/v1beta1`, `kind: SecretStore`**: Define o tipo de recurso.
-   **`metadata.name: solr-secret-store`, `metadata.namespace: solr`**: Nomeia e localiza o `SecretStore`.
-   **`spec.provider.vault`**: Especifica que o provedor de segredos é o Vault.
    -   **`server: http://vault.vault:8200`**: O endereço do servidor Vault dentro do cluster Kubernetes. O nome `vault.vault` sugere que o Vault está rodando no namespace `vault` e é acessível através de um serviço Kubernetes chamado `vault` na porta `8200`. O uso de `http` implica que a comunicação interna entre o ESO e o Vault pode não ser criptografada com TLS, ou que o TLS é tratado em outra camada (ex: service mesh). Para produção, HTTPS é recomendado.
    -   **`path: rancher`**: O caminho (mount path) do KV (Key-Value) Secrets Engine versão 2 no Vault onde os segredos relevantes para o Rancher (e, por extensão, para o Solr neste contexto) estão armazenados. Por exemplo, um segredo específico do Solr poderia estar em `rancher/data/solr-prod/credentials`.
    -   **`version: v2`**: Especifica o uso do KV Secrets Engine v2 do Vault, que suporta versionamento de segredos e uma estrutura de caminho ligeiramente diferente (com `/data/` inserido para acesso aos dados).
    -   **`auth.kubernetes`**: Configura o método de autenticação que o External Secrets Operator usará para se autenticar no Vault.
        -   **`mountPath: kubernetes`**: O caminho no Vault onde o método de autenticação Kubernetes está habilitado.
        -   **`role: solr`**: O nome da "role" de autenticação Kubernetes configurada no Vault. Esta role no Vault vincula um ServiceAccount do Kubernetes a políticas do Vault, definindo quais segredos o ServiceAccount pode acessar.
        -   **`serviceAccountRef.name: default`**: Especifica que o ServiceAccount chamado `default` no namespace `solr` (o mesmo namespace do `SecretStore`) será usado pelo External Secrets Operator para se autenticar no Vault. Este ServiceAccount precisa ter as permissões apropriadas no Vault, concedidas através da role `solr`.

**Pré-requisitos no Vault (Configuração Única):**
1.  O Vault deve estar instalado e operacional.
2.  O método de autenticação Kubernetes (`kubernetes`) deve estar habilitado no Vault e configurado para o cluster Kubernetes correto.
3.  Uma KV v2 secrets engine deve estar montada no caminho `rancher` (ou o caminho especificado em `spec.provider.vault.path`).
4.  Uma política (policy) no Vault deve ser criada, concedendo permissão de leitura aos segredos específicos do Solr (ex: `read` no caminho `rancher/data/solr-prod/credentials`).
5.  Uma role de autenticação Kubernetes (`solr`) deve ser criada no Vault, vinculando a política acima ao ServiceAccount `default` do namespace `solr` do cluster Kubernetes.

#### 7.2.3. Configuração do `ExternalSecret`: Sincronizando Segredos do Vault para o Kubernetes

O arquivo `apps/solr/solr-secret/solr-secret.yaml` define um recurso `ExternalSecret`. Este objeto instrui o External Secrets Operator a buscar um segredo específico do Vault (usando o `SecretStore` configurado) e criar ou atualizar um Segredo Kubernetes nativo com esses dados.

**Conteúdo e Análise do `apps/solr/solr-secret/solr-secret.yaml`:**

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: solr-external-secret
  namespace: solr
spec:
  secretStoreRef:
    name: solr-secret-store
    kind: SecretStore
  target:
    name: solr-usr-secret
  data:
    - remoteRef:
        key: prod # Assumindo que o segredo no Vault é 'prod' sob o path 'rancher/data/'
        property: solr_admin # A chave dentro do segredo 'prod' que contém a senha
      secretKey: solr-password # A chave no Segredo Kubernetes onde a senha será armazenada
```

-   **`apiVersion: external-secrets.io/v1beta1`, `kind: ExternalSecret`**: Define o tipo de recurso.
-   **`metadata.name: solr-external-secret`, `metadata.namespace: solr`**: Nomeia e localiza o `ExternalSecret`.
-   **`spec.secretStoreRef`**: Referencia o `SecretStore` a ser usado.
    -   **`name: solr-secret-store`**: Aponta para o `SecretStore` (`solr-secret-store`) definido anteriormente, que sabe como se conectar e autenticar no Vault.
    -   **`kind: SecretStore`**: Especifica o tipo do recurso referenciado.
-   **`spec.target`**: Define como o Segredo Kubernetes será criado ou atualizado.
    -   **`name: solr-usr-secret`**: Especifica o nome do Segredo Kubernetes que será gerenciado pelo External Secrets Operator. Este é o Segredo `solr-usr-secret` que a aplicação Solr (configurada no `values.yaml`, Seção 3.2) espera encontrar para obter suas credenciais.
-   **`spec.data`**: Define o mapeamento entre os dados no Vault e as chaves no Segredo Kubernetes.
    -   **`remoteRef`**: Especifica o segredo a ser buscado no Vault.
        -   **`key: prod`**: O nome do segredo (o caminho após o mount path do KV engine e o `/data/` prefixo do KV v2) a ser buscado no Vault. Se o `path` no `SecretStore` é `rancher` e a versão é `v2`, então o ESO buscará o segredo em `rancher/data/prod` no Vault.
        -   **`property: solr_admin`**: Especifica a chave *dentro* do segredo `prod` no Vault cujo valor será extraído. Ou seja, espera-se que o Vault contenha um segredo em `rancher/data/prod` que tenha uma chave chamada `solr_admin`, e o valor dessa chave é a senha real do Solr.
    -   **`secretKey: solr-password`**: Especifica o nome da chave sob a qual o valor da propriedade `solr_admin` (a senha) será armazenado no Segredo Kubernetes `solr-usr-secret`.

#### 7.2.4. Fluxo de Gerenciamento e Atualização de Segredos

1.  **Armazenamento Seguro no Vault:** A senha real do administrador do Solr é armazenada de forma segura no Vault, no caminho `rancher/data/prod` (ou similar, dependendo da estrutura exata no Vault) sob a chave `solr_admin`.
2.  **Sincronização Automática pelo ESO:** O External Secrets Operator monitora o recurso `ExternalSecret` (`solr-external-secret`).
    -   Ele usa o `SecretStore` (`solr-secret-store`) para se autenticar no Vault usando o ServiceAccount `default` do namespace `solr` e a role `solr` do Vault.
    -   Busca o valor da chave `solr_admin` do segredo `prod` no Vault.
    -   Cria ou atualiza o Segredo Kubernetes `solr-usr-secret` no namespace `solr`, colocando a senha recuperada na chave `solr-password`.
3.  **Consumo pela Aplicação Solr:** A implantação do Solr (via chart Helm, conforme Seção 3.2) está configurada para ler a chave `solr-password` do Segredo Kubernetes `solr-usr-secret` para obter a senha de autenticação.
4.  **Procedimento de Atualização de Senha (Manutenção):**
    a.  **Atualizar no Vault:** O administrador do sistema ou de segurança atualiza o valor da chave `solr_admin` no segredo `prod` diretamente na interface ou API do Vault.
    b.  **Sincronização Automática:** O External Secrets Operator detecta a mudança no Vault (geralmente em seu próximo intervalo de polling/sincronização, ou se o Vault notificar via webhooks, se configurado) e atualiza automaticamente o conteúdo do Segredo Kubernetes `solr-usr-secret`.
    c.  **Efetivação no Solr:** Os pods Solr, ao reiniciarem ou se forem capazes de recarregar dinamicamente suas configurações de autenticação (o que é menos comum para senhas de autenticação primária), começarão a usar a nova senha. Geralmente, um rolling restart dos pods Solr é a maneira mais segura de garantir que a nova senha seja efetivamente utilizada por todas as instâncias.

Este sistema robusto garante que as senhas e outras informações sensíveis não sejam armazenadas em texto plano nos arquivos de configuração do GitOps, aderindo às melhores práticas de segurança. Apenas a referência ao local seguro (Vault) e a mecânica de sincronização são definidas no Git, mantendo o próprio segredo fora do versionamento de código.

---


### 7.3. Servidor Dedicado para Upload de Plugins e Arquivos Adicionais (`apps/solr/solr-server-upload/`)

Para facilitar a adição de plugins Solr customizados ou outros arquivos de configuração que precisam ser disponibilizados para os pods Solr (especificamente no diretório `/solr-plugins/`, conforme configurado em `SOLR_OPTS` e `args`), um servidor de upload simples é implantado no ambiente. Este servidor permite que os usuários enviem arquivos para um volume compartilhado (NFS) que é montado pelos pods Solr.

As configurações para este servidor de upload estão localizadas no diretório `apps/solr/solr-server-upload/` do repositório GitOps.

#### 7.3.1. Orquestração do Servidor de Upload via Fleet e Kustomize

O arquivo `apps/solr/solr-server-upload/fleet.yaml` define como este componente é gerenciado pelo Fleet, utilizando Kustomize para definir os recursos Kubernetes.

**Conteúdo e Análise do `apps/solr/solr-server-upload/fleet.yaml`:**

```yaml
defaultNamespace: solr 
name: solr-server-upload
namespaceLabels:
  team: seint
kustomize:
  dir: ./kustomize
```

-   **`defaultNamespace: solr`**: Especifica que todos os recursos relacionados ao servidor de upload (Deployment, Service, Ingress, PVC) serão implantados no namespace `solr`.
-   **`name: solr-server-upload`**: Define o nome deste bundle do Fleet, identificando-o como o componente do servidor de upload do Solr.
-   **`namespaceLabels: {team: seint}`**: Aplica o rótulo `team: seint` ao namespace `solr`.
-   **`kustomize.dir: ./kustomize`**: Instrui o Fleet a usar o Kustomize para gerenciar os manifestos Kubernetes. Os arquivos de definição dos recursos estão localizados no subdiretório `kustomize/` (ou seja, `apps/solr/solr-server-upload/kustomize/`).

#### 7.3.2. Detalhamento dos Recursos Kubernetes Gerenciados pelo Kustomize

O arquivo `apps/solr/solr-server-upload/kustomize/kustomization.yaml` lista os manifestos YAML que definem os componentes do servidor de upload e aplica rótulos comuns a eles.

**Conteúdo e Análise do `kustomization.yaml`:**

```yaml
namespace: solr # Garante que os recursos sejam criados no namespace solr
resources:
  - plugin-folder-nfs-pvc.yaml
  - simple-upload-server-dep.yaml
  - simple-upload-server-ing.yaml
  - simple-upload-server-svc.yaml
commonLabels:
  app: solr-upload-plugin-server
  team: seint
```

-   **`namespace: solr`**: Embora o `fleet.yaml` já defina o `defaultNamespace`, esta linha no `kustomization.yaml` reforça que os recursos gerados ou referenciados por este Kustomization pertencem ao namespace `solr`.
-   **`resources:`**: Lista os arquivos de manifesto que o Kustomize processará e aplicará:
    -   `plugin-folder-nfs-pvc.yaml`: Define o PersistentVolumeClaim para o armazenamento dos plugins.
    -   `simple-upload-server-dep.yaml`: Define o Deployment do servidor de upload.
    -   `simple-upload-server-ing.yaml`: Define o Ingress para acesso externo ao servidor de upload.
    -   `simple-upload-server-svc.yaml`: Define o Service Kubernetes para o servidor de upload.
-   **`commonLabels:`**: Aplica os rótulos `app: solr-upload-plugin-server` e `team: seint` a todos os recursos criados por este Kustomization. Isso facilita a identificação, seleção e gerenciamento desses recursos no Kubernetes.

#### 7.3.3. Configuração do PersistentVolumeClaim (PVC) para a Pasta de Plugins (NFS)

O arquivo `apps/solr/solr-server-upload/kustomize/plugin-folder-nfs-pvc.yaml` define o PVC que representa o armazenamento compartilhado onde os plugins serão enviados e de onde o Solr os lerá.

**Conteúdo e Análise do `plugin-folder-nfs-pvc.yaml`:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: solr-plugin-folder
  namespace: solr
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-client
  resources:
    requests:
      storage: 1Gi
```

-   **`kind: PersistentVolumeClaim`, `name: solr-plugin-folder`**: Cria uma solicitação de armazenamento chamada `solr-plugin-folder` no namespace `solr`.
-   **`spec.accessModes: [ReadWriteMany]`**: (RWX) Crucial aqui. Significa que o volume pode ser montado como leitura-escrita por múltiplos nós (e, portanto, múltiplos pods) simultaneamente. Isso é necessário porque o pod do servidor de upload (que escreve os plugins) e os múltiplos pods Solr (que leem os plugins) precisam acessar este mesmo volume. Volumes NFS são um tipo comum de armazenamento que suporta RWX.
-   **`spec.storageClassName: nfs-client`**: Especifica que este PVC deve ser provisionado usando a `StorageClass` chamada `nfs-client`. Esta StorageClass é responsável por interagir com um provisionador NFS (como o `nfs-subdir-external-provisioner` ou similar) para criar dinamicamente um diretório no servidor NFS e o PersistentVolume correspondente.
-   **`spec.resources.requests.storage: 1Gi`**: Solicita 1 Gibibyte de espaço para este volume de plugins. Este é o mesmo PVC (`solr-plugin-folder`) que é referenciado e montado pelos pods Solr no caminho `/solr-plugins/` (conforme definido no `values.yaml` do Solr, Seção 3.6).

#### 7.3.4. Análise do Deployment do Servidor de Upload

O arquivo `apps/solr/solr-server-upload/kustomize/simple-upload-server-dep.yaml` define o Deployment que gerencia os pods do servidor de upload.

*(Nota: O conteúdo exato de `simple-upload-server-dep.yaml` não foi fornecido no fluxo de eventos, mas com base nos documentos anteriores e no nome, podemos inferir sua estrutura típica.)*

**Estrutura Esperada do `simple-upload-server-dep.yaml`:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: solr-upload-plugin-server
  namespace: solr
  # labels virão de commonLabels no kustomization.yaml
spec:
  replicas: 1 # Geralmente 1 réplica é suficiente para um servidor de upload
  selector:
    matchLabels:
      app: solr-upload-plugin-server # Deve corresponder aos labels do pod
  template:
    metadata:
      labels:
        app: solr-upload-plugin-server # Rótulo para o pod
        team: seint
    spec:
      containers:
      - name: solr-upload-plugin-server
        image: mayth/simple-upload-server:latest # Imagem do servidor de upload
        ports:
        - containerPort: 8080 # Porta que o servidor escuta
        args: # Argumentos para configurar o servidor de upload
          - "--host=0.0.0.0"
          - "--port=8080"
          - "--upload-path=/files" # Caminho interno ao contêiner para salvar uploads
          - "--token=true" # Habilita autenticação por token
          - "--max-size=1000000000" # Tamanho máximo de upload (1GB)
        env:
        - name: TOKEN # Variável de ambiente para o token
          value: "desenvolteste123" # TOKEN FIXO - RISCO DE SEGURANÇA
        volumeMounts:
        - name: plugin-storage
          mountPath: /files # Monta o PVC no upload-path do servidor
        # resources: requests e limits (devem ser definidos)
        # readinessProbe e livenessProbe (devem ser definidos)
      volumes:
      - name: plugin-storage
        persistentVolumeClaim:
          claimName: solr-plugin-folder # Usa o PVC definido anteriormente
```

-   **Imagem (`mayth/simple-upload-server:latest`):** Utiliza uma imagem pública de um servidor de upload simples. **Usar a tag `:latest` em produção não é uma boa prática**, pois pode levar a implantações inconsistentes se a imagem for atualizada sem controle. Uma tag de versão específica da imagem deveria ser usada.
-   **Argumentos (`args`):** Configuram o comportamento do servidor, como a porta de escuta (`8080`), o caminho de upload interno (`/files`), e a habilitação de autenticação por token.
-   **Token de Autenticação (`env.TOKEN: "desenvolteste123"`):** Define um token de autenticação fixo e diretamente no manifesto. **Este é um risco de segurança significativo.** Em um ambiente de produção, este token deveria ser gerenciado como um Segredo Kubernetes e injetado de forma segura, ou um método de autenticação mais robusto deveria ser usado.
-   **Montagem de Volume (`volumeMounts`):** Monta o PVC `solr-plugin-folder` (referenciado como `plugin-storage` no volume do pod) no caminho `/files` do contêiner. Este é o diretório onde o servidor de upload salvará os arquivos enviados.
-   **Recursos e Probes:** É crucial definir `resources` (requests e limits de CPU/memória) e `readinessProbe`/`livenessProbe` para o contêiner, garantindo seu bom funcionamento e gerenciamento pelo Kubernetes.

#### 7.3.5. Configuração do Ingress para Acesso Externo ao Servidor de Upload

O arquivo `apps/solr/solr-server-upload/kustomize/simple-upload-server-ing.yaml` define como o servidor de upload é exposto externamente.

*(Nota: O conteúdo exato não foi fornecido, mas podemos inferir.)*

**Estrutura Esperada do `simple-upload-server-ing.yaml`:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: solr-upload-plugin-server
  namespace: solr
  # annotations: (ex: para classe de ingress, reescritas, etc.)
spec:
  rules:
  - host: solr-upload.producao.rancher.tcu.gov.br # Hostname para acesso
    http:
      paths:
      - path: / # Caminho para o servidor de upload
        pathType: Prefix
        backend:
          service:
            name: solr-upload-plugin-server # Nome do Service do servidor de upload
            port:
              number: 8080 # Porta do Service
  # tls: (configuração TLS, se aplicável)
```

-   **Hostname (`solr-upload.producao.rancher.tcu.gov.br`):** Define a URL pela qual o servidor de upload será acessível. Uma entrada DNS correspondente é necessária.
-   **Backend Service:** Aponta para o Service `solr-upload-plugin-server` na porta `8080`.

#### 7.3.6. Configuração do Service Kubernetes para o Servidor de Upload

O arquivo `apps/solr/solr-server-upload/kustomize/simple-upload-server-svc.yaml` define o Service que expõe internamente os pods do servidor de upload.

*(Nota: O conteúdo exato não foi fornecido, mas podemos inferir.)*

**Estrutura Esperada do `simple-upload-server-svc.yaml`:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: solr-upload-plugin-server
  namespace: solr
spec:
  selector:
    app: solr-upload-plugin-server # Seleciona os pods do Deployment do servidor de upload
  ports:
  - protocol: TCP
    port: 8080 # Porta que o Service expõe
    targetPort: 8080 # Porta do contêiner onde o servidor está escutando
  # type: ClusterIP (padrão)
```

-   **Selector (`app: solr-upload-plugin-server`):** Garante que o Service direcione o tráfego para os pods corretos do servidor de upload.
-   **Ports:** Mapeia a porta `8080` do Service para a `targetPort: 8080` dos contêineres.

#### 7.3.7. Procedimento Detalhado para Upload de Plugins e Considerações de Segurança

1.  **Acesso ao Servidor:** O servidor de upload estará acessível via `http://solr-upload.producao.rancher.tcu.gov.br` (ou `https`, se o TLS for configurado no Ingress ou externamente).
2.  **Autenticação:** Para fazer um upload (ex: usando `curl` ou uma ferramenta de cliente HTTP), o token `desenvolteste123` precisará ser fornecido. A forma exata de passar o token (ex: header `Authorization: Bearer <token>`, ou um parâmetro de formulário) dependerá de como a imagem `mayth/simple-upload-server` implementa a autenticação por token.
    ```bash
    # Exemplo hipotético com curl, pode variar:
    curl -X POST -F "file=@/caminho/para/meu-plugin.jar" \
         -H "Authorization: Bearer desenvolteste123" \
         http://solr-upload.producao.rancher.tcu.gov.br/upload
    ```
3.  **Armazenamento:** Arquivos enviados para este servidor serão salvos no volume NFS compartilhado (montado via PVC `solr-plugin-folder`) no diretório que o servidor de upload usa como `upload-path` (ex: `/files`).
4.  **Disponibilidade para o Solr:** Uma vez que um arquivo JAR de plugin é enviado, ele se torna disponível no caminho `/solr-plugins/` dentro dos pods Solr (devido à montagem do mesmo PVC).
5.  **Carregamento pelo Solr:** Para que o Solr efetivamente carregue e utilize o novo plugin:
    a.  **Configuração do Solr:** Pode ser necessário configurar o Solr para reconhecer o novo plugin. Isso pode envolver a modificação de arquivos de configuração de um core/collection específico (ex: `solrconfig.xml` para adicionar a tag `<lib dir_ou_jar_path>`) ou o uso da API de Config do Solr para adicionar/atualizar componentes que usam o plugin.
    b.  **Recarregar Core/Collection ou Reiniciar Solr:** Após a configuração, o core/collection afetado geralmente precisa ser recarregado (via API Admin do Solr) para que o Solr detecte e carregue o novo plugin. Em alguns casos, ou para certos tipos de plugins, um reinício completo dos pods Solr pode ser necessário.

**Considerações de Segurança Urgentes:**
-   **Token Fixo e Fraco:** O token `desenvolteste123` é um risco de segurança crítico. Ele é fixo, visível no GitOps (se o `simple-upload-server-dep.yaml` for como o inferido) e trivial. Deve ser substituído por um token forte e gerenciado como um Segredo Kubernetes.
-   **Imagem `:latest`:** Usar uma tag de imagem específica em vez de `:latest` para `mayth/simple-upload-server`.
-   **Controle de Acesso à Rede:** O Ingress expõe o servidor de upload. Idealmente, o acesso a esta URL deveria ser restrito (ex: por IP de origem, VPN, ou autenticação no nível do Ingress Controller) para evitar uploads não autorizados.
-   **Validação de Uploads:** O servidor de upload simples pode não ter validação robusta dos tipos de arquivo ou conteúdo. Um atacante poderia tentar enviar arquivos maliciosos.
-   **Monitoramento e Auditoria:** Os uploads para este servidor devem ser monitorados e auditados.

Embora o servidor de upload forneça uma conveniência, suas implicações de segurança na configuração atual precisam ser cuidadosamente consideradas e mitigadas.

---


## 8. Configuração da Classe de Armazenamento (StorageClass) para Solr e Zookeeper (`apps/vsphere/sc-solr/`)

Tanto os pods Solr quanto os pods Zookeeper requerem armazenamento persistente para seus dados. Conforme visto nas Seções 3.9 (Persistência do Solr) e 3.11 (Persistência do Zookeeper), ambos estão configurados para usar uma `StorageClass` chamada `sc-solr`. Esta StorageClass define como os PersistentVolumes (PVs) serão dinamicamente provisionados no ambiente vSphere do TCU.

O diretório `apps/vsphere/sc-solr/` no GitOps contém os manifestos que definem e gerenciam esta StorageClass via Fleet.

### 8.1. Orquestração da StorageClass via Fleet

O arquivo `apps/vsphere/sc-solr/fleet.yaml` gerencia a implantação do recurso `StorageClass` no cluster Kubernetes.

**Conteúdo e Análise do `apps/vsphere/sc-solr/fleet.yaml`:**

```yaml
defaultNamespace: kube-system
name: rancher-vsphere-solr
```

-   **`defaultNamespace: kube-system`**
    -   **Descrição Detalhada:** Especifica que o recurso `StorageClass` definido neste bundle do Fleet será criado no namespace `kube-system`. StorageClasses são recursos de nível de cluster, mas sua definição pode ser colocada em um namespace específico para fins de gerenciamento pelo Fleet. O namespace `kube-system` é frequentemente usado para recursos de infraestrutura do cluster.

-   **`name: rancher-vsphere-solr`**
    -   **Descrição Detalhada:** Define o nome deste bundle do Fleet como `rancher-vsphere-solr`. Este nome é usado para identificar este conjunto de configurações de StorageClass na interface do Fleet no Rancher.

**Funcionamento da Orquestração:**
O Fleet monitora o diretório `apps/vsphere/sc-solr/`. Quando este `fleet.yaml` é processado, o Fleet aplica o manifesto `sc-solr.yaml` (que deve estar no mesmo diretório ou ser referenciado de outra forma, seguindo o padrão do Fleet de aplicar todos os YAMLs de um bundle) ao cluster Kubernetes. Isso cria ou atualiza a `StorageClass` `sc-solr`.

### 8.2. Definição Detalhada da StorageClass `sc-solr`

O arquivo `apps/vsphere/sc-solr/sc-solr.yaml` contém a definição do recurso `StorageClass`.

**Conteúdo e Análise do `apps/vsphere/sc-solr/sc-solr.yaml`:**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
  name: sc-solr
parameters:
  ## Datastore FA3_DSC8_VMFS_004
  datastoreURL: ds:///vmfs/volumes/65df3acd-d63c1685-31b5-84160c6ac040/
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
volumeBindingMode: Immediate
allowVolumeExpansion: true
```

-   **`apiVersion: storage.k8s.io/v1`, `kind: StorageClass`**: Define o tipo de recurso Kubernetes.
-   **`metadata.name: sc-solr`**: O nome da StorageClass, que é referenciado nas configurações de persistência do Solr e do Zookeeper (`persistence.storageClass: sc-solr`).
-   **`metadata.annotations`**: 
    -   **`storageclass.kubernetes.io/is-default-class: "false"`**: Garante que esta StorageClass não seja usada como a padrão para PVCs que não especificam explicitamente uma `storageClassName`. Isso é importante para evitar que outros workloads usem inadvertidamente este armazenamento configurado especificamente para o Solr.
-   **`provisioner: csi.vsphere.vmware.com`**
    -   **Descrição Detalhada:** Especifica que o provisionador de armazenamento a ser usado é o driver CSI (Container Storage Interface) do vSphere (`csi.vsphere.vmware.com`). Este driver permite que o Kubernetes interaja com a infraestrutura de armazenamento do VMware vSphere para provisionar e gerenciar volumes dinamicamente.
-   **`parameters:`**
    -   **`datastoreURL: ds:///vmfs/volumes/65df3acd-d63c1685-31b5-84160c6ac040/`**: Este é um parâmetro específico do provisionador vSphere CSI. Ele especifica o URL do datastore vSphere onde os discos virtuais (VMDKs) para os PersistentVolumes serão criados. O comentário `## Datastore FA3_DSC8_VMFS_004` indica o nome amigável deste datastore no ambiente vSphere. A URL `ds:///vmfs/volumes/65df3acd-d63c1685-31b5-84160c6ac040/` é o identificador único (UUID) do datastore VMFS.
    -   **Implicação:** Todos os volumes provisionados por esta StorageClass serão criados neste datastore específico. A escolha do datastore é crucial e deve considerar fatores como desempenho (ex: SSD vs. HDD), capacidade, e tiers de armazenamento.
-   **`reclaimPolicy: Retain`**
    -   **Descrição Detalhada:** Define o que acontece com o PersistentVolume (e o disco VMDK subjacente no vSphere) quando o PersistentVolumeClaim correspondente é excluído. `Retain` significa que o PV e seus dados não serão excluídos automaticamente. O PV entrará no estado `Released` e precisará ser recuperado ou excluído manualmente pelo administrador. Esta é uma política segura para dados críticos, como os do Solr, pois previne a perda acidental de dados. A alternativa comum é `Delete`, que excluiria o PV e o VMDK.
-   **`volumeBindingMode: Immediate`**
    -   **Descrição Detalhada:** Controla quando o provisionamento do PV e a vinculação ao PVC ocorrem.
        -   `Immediate`: O provisionamento do PV ocorre assim que o PVC é criado. Isso é simples, mas pode não ser ideal para clusters com múltiplas zonas de disponibilidade ou topologias complexas, pois o PV pode ser provisionado em uma zona onde o pod que o solicita não pode ser agendado.
        -   A alternativa é `WaitForFirstConsumer`, onde o provisionamento do PV é adiado até que um pod que usa o PVC seja agendado. O scheduler do Kubernetes então considera as restrições de topologia do pod ao decidir onde provisionar o PV. Para o Solr, que usa `nodeSelector` e `affinity` para direcionar pods a nós específicos, `Immediate` pode ser aceitável se os nós Solr tiverem acesso ao datastore especificado.
-   **`allowVolumeExpansion: true`**
    -   **Descrição Detalhada:** Permite que os PersistentVolumes provisionados por esta StorageClass sejam expandidos após a criação. Se um PVC criado com esta StorageClass precisar de mais espaço, seu tamanho solicitado pode ser aumentado, e o driver CSI do vSphere tentará expandir o VMDK subjacente e o sistema de arquivos (se suportado pelo driver e pelo sistema de arquivos). Esta é uma funcionalidade importante para gerenciar o crescimento dos dados do Solr ao longo do tempo sem a necessidade de migrações complexas.

**Implicações para o Ambiente Solr:**

-   **Desempenho e Confiabilidade:** O desempenho e a confiabilidade do armazenamento Solr dependem diretamente das características do datastore vSphere `FA3_DSC8_VMFS_004` e da configuração do driver vSphere CSI.
-   **Gerenciamento de Capacidade:** Com `allowVolumeExpansion: true`, os volumes podem ser aumentados. No entanto, o monitoramento da capacidade do datastore subjacente no vSphere ainda é essencial.
-   **Recuperação de Desastres:** A política `Retain` ajuda a proteger contra a exclusão acidental de dados, mas uma estratégia de backup completa (como os backups do Solr para NFS, descritos na Seção 7.1) ainda é fundamental para a recuperação de desastres.

Esta configuração de StorageClass demonstra uma integração específica com a infraestrutura vSphere do TCU, adaptando o provisionamento de armazenamento do Kubernetes às capacidades e recursos do ambiente de virtualização subjacente.

---


## 9. Sugestões de Melhoria e Otimização do Ambiente Solr

Com base na análise detalhada da configuração atual do Solr no ambiente Rancher do TCU, identificamos diversas oportunidades para aprimorar ainda mais a robustez, segurança, desempenho e manutenibilidade da plataforma. Estas sugestões visam otimizar a implantação existente, alinhando-a com as melhores práticas da indústria e as necessidades específicas do TCU.

### 9.1. Reforço da Segurança

-   **Revisão e Restrição da Política de Segurança Java (`security.policy`):**
    -   **Problema:** A política atual (`apps/solr/solr/security/security.policy`) contém permissões amplas (ex: `java.util.PropertyPermission "*", "read,write";`, `java.io.FilePermission` baseada em `solr.allowPaths=*`).
    -   **Sugestão:** Realizar uma auditoria detalhada da política para aplicar o princípio do menor privilégio. Listar explicitamente apenas as propriedades de sistema, arquivos e permissões de rede estritamente necessários para a operação do Solr e seus plugins. Remover permissões relacionadas a testes ou componentes não utilizados em produção. Considerar o uso de ferramentas de análise estática ou monitoramento de chamadas de segurança Java para identificar permissões excessivas.
    -   **Impacto Esperado:** Redução significativa da superfície de ataque em caso de exploração de vulnerabilidades.

-   **Habilitação e Configuração de `NetworkPolicy` para Solr e Zookeeper:**
    -   **Problema:** Tanto o Solr (`networkPolicy.enabled: false` no `values.yaml`) quanto o Zookeeper (`zookeeper.networkPolicy.enabled: false`) não possuem políticas de rede específicas definidas pelo chart Helm.
    -   **Sugestão:** Habilitar e configurar `NetworkPolicy` para ambos. Para o Solr, restringir o tráfego de entrada para as fontes necessárias (Ingress Controller, pods da aplicação, outros nós Solr para comunicação interna) e o tráfego de saída (Zookeeper, sistema de backup, Vault). Para o Zookeeper, permitir entrada apenas dos pods Solr e saída apenas para outros nós Zookeeper (se aplicável).
    -   **Impacto Esperado:** Isolamento de rede aprimorado, limitando o movimento lateral de potenciais atacantes.

-   **Segurança do Servidor de Upload de Plugins (`solr-server-upload`):**
    -   **Problema:** O token de autenticação (`desenvolteste123`) é fixo, fraco e visível no manifesto (se a inferência do `simple-upload-server-dep.yaml` estiver correta). A imagem `:latest` é usada.
    -   **Sugestão:** 
        1.  Substituir o token fixo por um token forte, gerado aleatoriamente e gerenciado como um Segredo Kubernetes, injetado via variável de ambiente no pod do servidor de upload.
        2.  Utilizar uma tag de versão específica para a imagem `mayth/simple-upload-server` ou considerar uma alternativa mais robusta e segura para o servidor de upload, possivelmente com integração a um sistema de autenticação mais forte (ex: OAuth2/OIDC).
        3.  Restringir o acesso à URL do servidor de upload no nível do Ingress Controller (ex: por IP de origem, VPN) ou adicionar autenticação mais forte.
        4.  Implementar logging e auditoria detalhados para todas as operações de upload.
    -   **Impacto Esperado:** Redução drástica do risco de uploads não autorizados ou maliciosos.

-   **Comunicação Segura (TLS) Interna:**
    -   **Problema:** A comunicação entre o External Secrets Operator e o Vault (`server: http://vault.vault:8200`) e potencialmente entre outros componentes internos pode não estar usando TLS.
    -   **Sugestão:** Garantir que toda a comunicação interna sensível, especialmente com o Vault, utilize HTTPS. Isso pode envolver a configuração de certificados TLS para os serviços internos ou a utilização de uma service mesh (como Istio ou Linkerd) para impor mTLS (mutual TLS) automaticamente.
    -   **Impacto Esperado:** Proteção contra sniffing de dados na rede interna do cluster.

### 9.2. Otimização de Desempenho e Recursos

-   **Ajuste Fino da Memória Java (`javaMem`) e Recursos do Pod Solr:**
    -   **Problema:** O `-Xmx100g` para o heap Solr é muito alto em comparação com o `requests.memory: 20Gi` e a ausência de `limits.memory` no pod. Isso cria um risco de OOMKilled pelo sistema operacional se o Solr tentar usar mais memória do que o nó pode fornecer, ou pelo Kubernetes se um `LimitRange` no namespace for mais restritivo.
    -   **Sugestão:** 
        1.  Realizar um profiling de memória do Solr sob carga representativa para determinar o uso real de heap e off-heap.
        2.  Ajustar `-Xms` e `-Xmx` para valores realistas e consistentes com os recursos do nó.
        3.  Definir `resources.limits.memory` para os pods Solr para um valor ligeiramente acima do `-Xmx` mais uma margem para off-heap e outros processos no contêiner (ex: `-Xmx60g`, `limits.memory: 70Gi`, assumindo que os nós têm essa capacidade e que `requests.memory` seja ajustado para algo como `50Gi` ou `60Gi` para uma QoS mais próxima de `Guaranteed`).
        4.  Monitorar de perto as pausas de Garbage Collection (GC) e ajustar os parâmetros do GC se necessário.
    -   **Impacto Esperado:** Maior estabilidade dos pods Solr, melhor previsibilidade do uso de recursos e redução do risco de OOMKilled.

-   **Otimização de Índices e Collections:**
    -   **Sugestão:** Implementar um ciclo regular de otimização de índices (merge de segmentos) para as collections, especialmente aquelas com alta taxa de escrita/atualização. Isso pode ser agendado via API do Solr (com cuidado para não impactar o desempenho em horários de pico).
    -   Analisar o schema das collections (`schema.xml` ou managed schema) para garantir que os tipos de campo, indexação e armazenamento sejam os mais eficientes para os padrões de consulta.
    -   Avaliar a configuração de shards e réplicas por collection para balancear desempenho de busca, resiliência e capacidade de escrita.
    -   **Impacto Esperado:** Melhor desempenho de busca, menor uso de disco e CPU.

-   **Cache do Solr:**
    -   **Sugestão:** Monitorar ativamente as estatísticas dos caches do Solr (filterCache, queryResultCache, documentCache, etc.) via JMX/Prometheus. Ajustar seus tamanhos e políticas de evicção com base nos padrões de uso para maximizar as taxas de acerto e minimizar o impacto no heap.
    -   **Impacto Esperado:** Redução da latência de consulta.

### 9.3. Melhorias na Manutenibilidade e Operação

-   **Gerenciamento de Espaço de Backup:**
    -   **Problema:** Os CronJobs de backup armazenam dados no NFS, mas não há um mecanismo automático de rotação ou exclusão de backups antigos definido no GitOps.
    -   **Sugestão:** Implementar um script ou CronJob separado para gerenciar a retenção de backups no NFS. Este script poderia excluir backups mais antigos que um determinado período (ex: manter backups diários por X dias, semanais por Y semanas, etc.), garantindo que o espaço no NFS não se esgote.
    -   **Impacto Esperado:** Prevenção de falhas de backup devido à falta de espaço e controle sobre os custos de armazenamento.

-   **Versionamento de Imagens Docker:**
    -   **Problema:** A imagem do servidor de upload (`mayth/simple-upload-server:latest`) usa a tag `:latest`.
    -   **Sugestão:** Sempre utilizar tags de versão específicas para todas as imagens Docker (Solr, Zookeeper, servidor de upload, etc.) nos manifestos do GitOps. Isso garante implantações reprodutíveis e controladas.
    -   **Impacto Esperado:** Maior estabilidade e previsibilidade nas implantações e atualizações.

-   **Documentação de Procedimentos de Restauração:**
    -   **Sugestão:** Documentar formalmente o procedimento de restauração de uma collection Solr a partir dos backups armazenados no NFS. Incluir os comandos da API Solr necessários e quaisquer passos de verificação.
    -   **Impacto Esperado:** Redução do tempo de recuperação (RTO) em caso de necessidade de restauração.

-   **Revisão do `volumeBindingMode` para StorageClass:**
    -   **Problema:** A `StorageClass` `sc-solr` usa `volumeBindingMode: Immediate`.
    -   **Sugestão:** Avaliar se `volumeBindingMode: WaitForFirstConsumer` seria mais apropriado, especialmente se houver planos de expandir o cluster Solr para múltiplas zonas de disponibilidade ou se a topologia dos nós se tornar mais complexa. `WaitForFirstConsumer` garante que o PV seja provisionado levando em conta as restrições de agendamento do pod.
    -   **Impacto Esperado:** Maior flexibilidade e otimização no provisionamento de volumes em topologias de cluster avançadas.

### 9.4. Monitoramento e Alertas Proativos

-   **Expandir Cobertura de Alertas:**
    -   **Sugestão:** Além das métricas básicas, configurar alertas no Prometheus/Alertmanager para condições críticas específicas do Solr, como:
        -   Latência de consulta elevada (percentis p95, p99).
        -   Taxa de erro de consulta alta.
        -   Baixo número de réplicas ativas por shard.
        -   Caches com baixa taxa de acerto.
        -   Uso de heap da JVM próximo ao máximo.
        -   Filas de atualização longas.
        -   Falhas de comunicação com o Zookeeper.
        -   Espaço em disco baixo nos volumes persistentes.
    -   **Impacto Esperado:** Detecção precoce de problemas, permitindo ação corretiva antes que afetem os usuários.

-   **Logging Centralizado e Análise:**
    -   **Sugestão:** Garantir que os logs do Solr e do Zookeeper sejam enviados para um sistema de logging centralizado (ex: ELK Stack, Loki/Grafana). Configurar dashboards e alertas com base nos logs para identificar erros e padrões anormais.
    -   **Impacto Esperado:** Facilidade na depuração de problemas e análise de causa raiz.

Ao implementar estas sugestões, o TCU pode evoluir seu ambiente Solr para um estado ainda mais otimizado, seguro e fácil de gerenciar, garantindo que ele continue a servir como uma plataforma de busca robusta e eficiente para suas aplicações.

---


## 10. Troubleshooting Básico e Coleta de Logs

Mesmo com uma configuração robusta, problemas podem ocorrer no ambiente Solr. Saber como diagnosticar e coletar informações relevantes é crucial para uma resolução rápida.

### 10.1. Verificando Status dos Pods e Recursos Kubernetes

A primeira etapa em qualquer troubleshooting no Kubernetes é verificar o status dos pods e outros recursos relacionados ao Solr e Zookeeper.

-   **Verificar Pods Solr e Zookeeper:**
    ```bash
    kubectl get pods -n solr -l app=solr # Para pods Solr
    kubectl get pods -n solr -l app=zookeeper # Para pods Zookeeper
    kubectl get pods -n solr -l app=solr-upload-plugin-server # Para o servidor de upload
    ```
    Procure por pods que não estejam no estado `Running` ou que tenham um alto número de `RESTARTS`.

-   **Descrever Pods com Problemas:**
    Se um pod estiver com problemas (ex: `CrashLoopBackOff`, `Error`, `Pending`), use `describe` para obter mais detalhes sobre seus eventos e condições:
    ```bash
    kubectl describe pod <nome-do-pod-com-problema> -n solr
    ```
    Isso pode revelar problemas como falhas ao puxar a imagem, erros de montagem de volume, falhas em readiness/liveness probes, ou OOMKilled.

-   **Verificar Logs dos Contêineres:**
    Os logs dos contêineres são a fonte primária de informação para erros da aplicação.
    ```bash
    kubectl logs <nome-do-pod-solr> -n solr -c solr # Logs do contêiner Solr principal
    kubectl logs <nome-do-pod-solr> -n solr -c metrics # Se houver um sidecar de métricas
    kubectl logs <nome-do-pod-zookeeper> -n solr # Logs do Zookeeper
    ```
    Para pods com múltiplos contêineres (como Solr com um exportador de métricas), especifique o contêiner com `-c <nome-do-contêiner>`.
    Se um pod estiver reiniciando constantemente, use a flag `-p` (ou `--previous`) para ver os logs da instância anterior que falhou:
    ```bash
    kubectl logs -p <nome-do-pod-com-problema> -n solr
    ```

-   **Verificar CronJobs de Backup:**
    ```bash
    kubectl get cronjobs -n solr # Lista todos os CronJobs de backup
    kubectl get jobs -n solr # Lista os Jobs criados pelos CronJobs
    ```
    Se um Job de backup falhar, verifique os logs do pod criado por aquele Job:
    ```bash
    kubectl logs <nome-do-pod-do-job-de-backup> -n solr
    ```

-   **Verificar ExternalSecrets e SecretStore:**
    ```bash
    kubectl get secretstore solr-secret-store -n solr -o yaml
    kubectl get externalsecret solr-external-secret -n solr -o yaml
    ```
    Verifique o status e os eventos para garantir que o External Secrets Operator está se comunicando com o Vault e sincronizando os segredos corretamente.

### 10.2. Acessando a Interface de Administração do Solr

A interface de administração do Solr (Solr Admin UI), acessível via `http://solr.producao.rancher.tcu.gov.br/`, é uma ferramenta poderosa para diagnóstico.

-   **Página de Logging:** A UI do Solr possui uma seção de Logging onde é possível visualizar logs recentes e até mesmo alterar níveis de log para classes específicas dinamicamente (útil para depuração).
-   **Status dos Cores/Collections:** Verifique o status de cada core/collection, incluindo o número de documentos, o tamanho do índice e se todas as réplicas estão ativas e sincronizadas.
-   **Análise de Consultas:** A UI permite executar consultas e analisar seus resultados e desempenho.
-   **Cloud -> ZK Status:** Para SolrCloud, a seção Cloud mostra o status da conexão com o Zookeeper e permite navegar pela estrutura de dados no Zookeeper.

### 10.3. Monitoramento com Prometheus e Grafana

Conforme configurado na Seção 3.10, as métricas do Solr são expostas ao Prometheus. Dashboards no Grafana (ou outra ferramenta de visualização) devem ser usados para monitorar:
-   **Métricas da JVM:** Uso de heap, atividade do Garbage Collector, contagem de threads.
-   **Métricas de Cache do Solr:** Taxas de acerto/erro, evicções, tamanho dos caches (filterCache, queryResultCache, documentCache).
-   **Desempenho de Query Handlers:** Requisições por segundo, latência média e percentil, taxa de erros.
-   **Operações de Atualização:** Taxa de commits, tempo de otimização.
-   **Status do SolrCloud:** Número de nós ativos, estado das collections, shards e réplicas.

Alertas configurados no Alertmanager com base nessas métricas ajudarão a identificar problemas proativamente.

### 10.4. Problemas Comuns e Dicas

-   **Solr não Inicia ou Pods em CrashLoopBackOff:**
    -   Verifique os logs do pod para exceções Java.
    -   Confirme se os volumes persistentes (PVCs) estão corretamente vinculados e montados.
    -   Verifique se o Zookeeper está acessível e saudável (para SolrCloud).
    -   Problemas de permissão em volumes montados (NFS) podem ser uma causa; o Init Container (Seção 3.7) deve lidar com isso, mas verifique seus logs se houver suspeita.
    -   Configurações incorretas no `values.yaml` ou `SOLR_OPTS`.

-   **Alto Uso de CPU/Memória:**
    -   Consultas ineficientes ou muito complexas.
    -   Configuração de cache inadequada.
    -   Heap da JVM mal dimensionado ou problemas de GC.
    -   Indexação intensiva.
    -   Use a UI do Solr e métricas do Prometheus para identificar gargalos.

-   **Consultas Lentas:**
    -   Verifique os logs do Solr para consultas lentas (se o logging estiver configurado para isso).
    -   Analise os planos de execução das consultas.
    -   Otimize os schemas e a configuração dos caches.
    -   Verifique se há gargalos de I/O no armazenamento persistente.

-   **Problemas de Comunicação com Zookeeper:**
    -   Verifique a conectividade de rede entre os pods Solr e Zookeeper.
    -   Verifique os logs dos pods Zookeeper.
    -   Garanta que o `jute.maxbuffer` esteja configurado adequadamente tanto no Solr quanto no Zookeeper se houver configurações grandes.

-   **Falhas de Backup:**
    -   Verifique os logs do pod do Job de backup.
    -   Confirme se o Solr tem permissão de escrita no local de backup (NFS).
    -   Verifique se a API de Backup do Solr está sendo chamada corretamente (URL, autenticação, parâmetros).
    -   Espaço insuficiente no destino do backup (NFS).

Lembre-se que a abordagem GitOps permite rastrear mudanças na configuração. Se um problema surgir após uma alteração, revisar os commits recentes no Git pode ajudar a identificar a causa.

---

## 11. Glossário de Termos

-   **Solr:** Uma plataforma de busca de código aberto baseada no Apache Lucene.
-   **SolrCloud:** Modo de operação distribuída do Solr, que fornece alta disponibilidade, tolerância a falhas e escalabilidade.
-   **Rancher:** Uma plataforma de gerenciamento de contêineres Kubernetes.
-   **Kubernetes (K8s):** Um sistema de orquestração de contêineres de código aberto.
-   **GitOps:** Uma metodologia para gerenciamento de infraestrutura e aplicações Kubernetes, onde o Git é a fonte única da verdade.
-   **Fleet:** Uma ferramenta do Rancher para gerenciamento GitOps de clusters e aplicações Kubernetes.
-   **Helm:** Um gerenciador de pacotes para Kubernetes, que usa "charts" para definir, instalar e atualizar aplicações.
-   **Chart Helm:** Um pacote contendo todos os recursos Kubernetes necessários para rodar uma aplicação.
-   **`values.yaml`:** Arquivo de configuração usado pelos charts Helm para customizar uma implantação.
-   **Kustomize:** Uma ferramenta para customizar manifestos Kubernetes sem modificar os originais.
-   **Pod:** A menor unidade de implantação no Kubernetes; um ou mais contêineres compartilhando recursos.
-   **Service:** Um recurso Kubernetes que expõe um conjunto de pods como um serviço de rede.
-   **Ingress:** Um recurso Kubernetes que gerencia o acesso externo (HTTP/HTTPS) aos serviços dentro do cluster.
-   **Deployment:** Um recurso Kubernetes que gerencia a implantação e atualização de pods stateless.
-   **StatefulSet:** Um recurso Kubernetes para gerenciar aplicações com estado, fornecendo armazenamento e identidades de rede estáveis.
-   **PersistentVolume (PV):** Um pedaço de armazenamento no cluster que foi provisionado por um administrador ou dinamicamente.
-   **PersistentVolumeClaim (PVC):** Uma solicitação de armazenamento feita por um usuário/pod.
-   **StorageClass:** Define diferentes "classes" de armazenamento (ex: rápido, lento) e como os PVs são provisionados.
-   **CSI (Container Storage Interface):** Um padrão para expor sistemas de armazenamento de blocos e arquivos para cargas de trabalho em contêineres.
-   **NFS (Network File System):** Um protocolo de sistema de arquivos distribuído.
-   **CronJob:** Um recurso Kubernetes que cria Jobs em uma programação recorrente.
-   **Job:** Um recurso Kubernetes que cria um ou mais pods e garante que um número especificado deles termine com sucesso.
-   **Secret:** Um recurso Kubernetes para armazenar informações sensíveis, como senhas ou chaves de API.
-   **External Secrets Operator (ESO):** Uma ferramenta que sincroniza segredos de provedores externos (como Vault) para Segredos Kubernetes.
-   **Vault (HashiCorp Vault):** Uma ferramenta para gerenciar segredos e proteger dados sensíveis.
-   **SecretStore:** Um recurso CRD do ESO que define como se conectar a um provedor de segredos externo.
-   **ExternalSecret:** Um recurso CRD do ESO que define qual segredo buscar e como mapeá-lo para um Segredo Kubernetes.
-   **Zookeeper (Apache Zookeeper):** Um serviço de coordenação distribuída usado pelo SolrCloud.
-   **JVM (Java Virtual Machine):** O ambiente de tempo de execução para aplicações Java, como o Solr.
-   **Java Security Manager:** Um componente da JVM que permite a definição de políticas de segurança granulares.
-   **`security.policy`:** Arquivo que define as permissões para o Java Security Manager.
-   **JMX (Java Management Extensions):** Uma tecnologia Java para monitorar e gerenciar aplicações Java.
-   **Prometheus:** Um sistema de monitoramento e alerta de código aberto.
-   **ServiceMonitor:** Um recurso CRD do Prometheus Operator que define como o Prometheus deve descobrir e raspar métricas.
-   **Collection:** No SolrCloud, uma coleção é um índice lógico que pode ser dividido em múltiplos shards e replicado.
-   **Shard:** Uma fatia horizontal de uma collection SolrCloud.
-   **Replica:** Uma cópia de um shard.
-   **NodeSelector:** Uma forma de restringir pods para serem agendados apenas em nós Kubernetes com rótulos específicos.
-   **Tolerations:** Permitem que pods sejam agendados em nós com "taints" correspondentes.
-   **Affinity/Anti-Affinity:** Regras que controlam como os pods são agendados em relação a outros pods ou nós.
-   **OCI (Open Container Initiative):** Um conjunto de padrões para formatos de contêiner e runtime.

---

## 12. Conclusão

A implantação do Solr no ambiente Rancher do TCU, gerenciada via GitOps com Fleet, Helm e Kustomize, representa uma abordagem moderna e robusta para operar uma plataforma de busca crítica. A análise detalhada dos arquivos de configuração revelou uma arquitetura bem pensada, que incorpora diversos componentes para garantir funcionalidade, segurança, persistência de dados e manutenibilidade.

Os principais aspectos abordados nesta documentação incluem:

-   **Configuração Centralizada:** O uso do `values.yaml` para o chart Helm do Solr permite um controle granular sobre todos os aspectos da implantação, desde a imagem Docker, alocação de recursos, autenticação, até a configuração do Zookeeper e integração com Prometheus.
-   **Gerenciamento GitOps:** O Fleet orquestra a aplicação das configurações do Solr, seus backups, segredos e o servidor de upload de plugins, garantindo que o estado do cluster reflita o que está definido no repositório Git.
-   **Segurança em Camadas:** A utilização do Java Security Manager com uma política customizada, a integração com HashiCorp Vault via External Secrets Operator para gerenciamento de senhas, e a configuração de backups automatizados são pilares importantes da estratégia de segurança. No entanto, foram identificadas oportunidades de reforço, como a restrição da política Java, a implementação de NetworkPolicies e o aprimoramento da segurança do servidor de upload.
-   **Persistência e Armazenamento:** A definição de uma `StorageClass` específica para o vSphere (`sc-solr`) e o uso de PVCs para Solr, Zookeeper e o volume de plugins NFS garantem a persistência dos dados e a flexibilidade para expansão.
-   **Manutenção e Operação:** Procedimentos para backup são automatizados via CronJobs. O servidor de upload de plugins, embora funcional, requer atenção à segurança. A documentação dos procedimentos de manutenção, como atualizações (inerentes ao GitOps) e restauração, é vital.

As sugestões de melhoria apresentadas na Seção 9 oferecem um roteiro para otimizar ainda mais o ambiente Solr, focando em segurança, desempenho, resiliência e manutenibilidade. A implementação dessas sugestões, juntamente com um monitoramento proativo e uma cultura de revisão contínua das configurações, permitirá ao TCU manter uma plataforma Solr de alta performance e confiabilidade.

Este documento serve como um guia detalhado para entender a configuração atual e como realizar a manutenção do ambiente Solr no Rancher. Espera-se que ele seja um recurso valioso para a equipe SEINT e outros administradores responsáveis pela plataforma, facilitando a operação diária, o troubleshooting e o planejamento futuro.

---

