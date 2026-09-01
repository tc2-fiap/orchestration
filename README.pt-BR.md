[English](README.en-US.md) · **Português**

# FIAP Games — Orchestration

O chart Helm guarda-chuva do sistema distribuído: infraestrutura compartilhada (PostgreSQL, RabbitMQ), o Ingress e todos os subcharts de serviço.

## Início rápido

```bash
kind create cluster --config kind/cluster-config.yaml
helm dependency update
helm install fiap-games .
```

Veja `../documentation/getting-started/GETTING_STARTED.pt-BR.md` se você já tem o repositório `documentation` clonado como irmão, ou [github.com/tc2-fiap/documentation](https://github.com/tc2-fiap/documentation/blob/main/getting-started/GETTING_STARTED.pt-BR.md) ([English](https://github.com/tc2-fiap/documentation/blob/main/getting-started/GETTING_STARTED.en-US.md)) caso contrário, para a lista completa de pré-requisitos, os passos de verificação e um passo a passo completo de demonstração (cadastro, compra de um jogo, observar `Pending → Paid`, trilha de auditoria de admin).

## Inspecionar o sistema em execução

Tudo vive no namespace `fiap-games`:

```bash
kubectl get pods -n fiap-games -o wide     # todo pod: status, node, IP, reinícios
kubectl get all -n fiap-games              # + Deployments, Services, ReplicaSets
kubectl get pods -n fiap-games -w          # ao vivo, atualiza conforme os pods sobem/reiniciam
```

Espere 8 pods quando tudo estiver de pé: Postgres, RabbitMQ, os cinco serviços de backend e o frontend — veja `GETTING_STARTED.md` §4 para o que é um resultado saudável vs. não saudável. Cada um tem o label `app=<nome-do-serviço>` (`app=orders-api`, `app=frontend`, etc.), que também é a forma mais rápida de seguir os logs de um serviço ou entrar no shell dele sem digitar o nome completo do pod (o hash no final muda a cada reinício/redeploy):

```bash
kubectl logs -n fiap-games -l app=orders-api --tail=100 -f
kubectl describe pod -n fiap-games -l app=payments-api    # detalhe completo: imagem, env, eventos, motivo do reinício
kubectl exec -n fiap-games -it deploy/orders-api -- sh
```

Se tiver o [`k9s`](https://k9scli.io/) instalado, `k9s -n fiap-games` dá um dashboard de terminal ao vivo sobre tudo isso — pods, logs e acesso a shell em uma única tela, sem comandos separados.

## O que tem aqui

```
Chart.yaml        # chart guarda-chuva — depende dos seis subcharts de serviço
values.yaml        # roles/schemas do Postgres, RabbitMQ, segredo JWT, seed de admin, chave do Resend
templates/          # namespace, Postgres, RabbitMQ, Ingress, Secrets de admin/resend
kind/               # configuração do cluster local
charts/             # arquivos de subchart resolvidos (gerados por `helm dependency update`)
```

Cada repositório de backend e o frontend possuem seu próprio subchart sob seu próprio diretório `k8s/`; este repositório apenas os compõe.

## Renomeando o namespace

Todo manifest — `templates/namespace.yaml` e o `Deployment`/`Service`/`ConfigMap` de cada serviço nos seis subcharts, além do Ingress — monta seu `namespace:` a partir de um único lugar, o `global.namespace` do `values.yaml` (padrão `fiap-games`). Basta mudar esse valor e reinstalar:

```yaml
# values.yaml
global:
  namespace: meu-novo-namespace
```

```bash
helm uninstall fiap-games -n fiap-games   # o namespace antigo não é renomeado nem apagado automaticamente
helm install fiap-games . -n meu-novo-namespace --create-namespace
```

Nenhum outro arquivo precisa ser editado — todo subchart já lê `{{ .Values.global.namespace }}` em vez de deixar o valor fixo. Duas outras coisas também se chamam `fiap-games`, mas são independentes desse valor e não são afetadas ao alterá-lo: o nome do **cluster kind** (`kind/cluster-config.yaml`) e o nome do **release/chart Helm** (`Chart.yaml`, o `fiap-games` em `helm install fiap-games .` acima) — renomeie essas duas também, separadamente, só se quiser consistência total de nomes.

## Documentação

A documentação completa do projeto — especificação, registro de decisões, arquitetura e este guia de primeiros passos — vive no repositório `documentation`, um repositório próprio: [`github.com/tc2-fiap/documentation`](https://github.com/tc2-fiap/documentation) (ou `../documentation/` se você o tiver clonado como irmão) — não neste repositório (`notes.md` 34, 44, 49).

## Contexto

Projeto acadêmico (FIAP). Veja [`README.pt-BR.md`](https://github.com/tc2-fiap/documentation/blob/main/README.pt-BR.md) no repositório `documentation` para a visão completa dos oito repositórios.
