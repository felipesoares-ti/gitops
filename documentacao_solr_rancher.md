# Documentação Completa: Instalação e Manutenção do Ambiente SOLR no Rancher

Esta documentação foi desenvolvida com base na análise dos ambientes Kubernetes gerenciados pelo Rancher no TCU, incluindo práticas de GitOps, automações e configurações específicas que garantem a segurança, eficiência e escalabilidade do Apache Solr.

## Índice
1. Introdução
2. Visão Geral do Ambiente Solr no Rancher
3. Procedimentos de Instalação e Configuração Principal do Solr
4. Gerenciamento da Aplicação Solr via Fleet
5. Gerenciamento da Política de Segurança
6. Manutenção do Ambiente Solr
7. Configuração de StorageClass para Solr e Zookeeper
8. Sugestões de Melhoria
9. Troubleshooting e Coleta de Logs
10. Glossário
11. Conclusão

---

## 1. Introdução
O Apache Solr é uma plataforma de busca poderosa e flexível, amplamente utilizada para indexação de dados e pesquisa eficiente. Este documento apresenta as melhores práticas para instalação e manutenção do Solr utilizando Rancher e GitOps, com foco nos requisitos e padrões do TCU.

---

## 2. Visão Geral do Ambiente Solr no Rancher
Por que isso é importante? O Solr no Rancher permite uma arquitetura escalável e simplificada, ideal para operações críticas. Aqui você encontrará uma visão detalhada de como o ambiente é estruturado.

---

## 3. Procedimentos de Instalação e Configuração Principal do Solr

### 3.1. Configurações Globais da Aplicação
As configurações globais definem parâmetros essenciais, como memória Java (`javaMem`), persistência de dados e políticas de rede. Por exemplo:
```yaml
javaMem: '-Xms16g -Xmx100g'
persistence:
  enabled: true
  mountPath: /bitnami/solr
  size: 1000Gi
```

### 3.2. Autenticação
A autenticação é configurada usando segredos existentes (`solr-usr-secret`). Isso garante que apenas usuários autorizados acessem o ambiente.

### 3.3. Otimizações Java
Inclua otimizações como:
```yaml
SOLR_OPTS: '-Dsolr.sharedLib=/solr-plugins/ -Djute.maxbuffer=5000000'
```

---

## 4. Gerenciamento da Aplicação Solr via Fleet
Utilizamos o Fleet para gerenciar a implantação do Solr. Esse processo automatiza atualizações e facilita a integração contínua (CI/CD).

---

## 5. Gerenciamento da Política de Segurança
A segurança é um aspecto crítico. Segredos são gerenciados usando o Vault e o External Secrets para garantir confidencialidade e integridade.

---

## 6. Manutenção do Ambiente Solr

### 6.1. Backups Automatizados
Backups são realizados por meio de CronJobs configurados para criar snapshots regulares das collections. Exemplo:
```yaml
containers:
  - command:
      - curl -u admin:${PASS} --location "http://${DOMINIO}/solr/admin/collections?action=BACKUP&collection=${COLLECTION}&location=${LOCATION}&name=${NAME}"
```

### 6.2. Upload de Plugins
Configuração de um servidor dedicado para uploads:
```yaml
containers:
  - name: solr-upload-plugin-server
    image: mayth/simple-upload-server:latest
    args:
      - '-document_root'
      - '/files'
```

---

## 7. Configuração de StorageClass para Solr e Zookeeper
A StorageClass `sc-solr` facilita a alocação dinâmica de volumes persistentes:
```yaml
storageClass: sc-solr
```

---

## 8. Sugestões de Melhoria
### 8.1. Reforço da Segurança
Considere habilitar políticas de rede e TLS para proteger o tráfego.

### 8.2. Melhorias na Manutenibilidade
Automatize tarefas repetitivas e implemente alertas proativos.

---

## 9. Troubleshooting e Coleta de Logs
### 9.1. Monitoramento
Integre Prometheus para métricas e Grafana para visualizações.

### 9.2. Problemas Comuns
Dicas práticas:
- Verifique se os pods estão em execução com `kubectl get pods`.
- Confira logs com `kubectl logs <pod-name>`.

---

## 10. Glossário
Inclua definições para termos técnicos como "StorageClass", "Ingress", e "CronJob".

---

## 11. Conclusão
Este documento fornece um guia prático para instalar e manter o Apache Solr no Rancher. Ao seguir essas instruções, você garante um ambiente seguro, eficiente e escalável.
