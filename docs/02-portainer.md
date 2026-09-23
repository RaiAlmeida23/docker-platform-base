# Portainer: conceitos, instalação, comandos e boas práticas

## Parte 1: O que é o Portainer
O Portainer é uma plataforma de gerenciamento que oferece uma interface gráfica (GUI) para administrar ambientes Docker e Kubernetes. Ele simplifica tarefas que normalmente exigiriam comandos no terminal, permitindo visualizar, criar e gerenciar contêineres, imagens, volumes, redes e stacks de forma intuitiva.

Com o Portainer, administradores podem acompanhar o uso de recursos, monitorar serviços e configurar aplicações em contêineres de maneira mais rápida e acessível, facilitando a gestão da infraestrutura. Nesta plataforma, ele é a camada principal de gerenciamento e o ponto de implantação de todos os demais serviços (Nginx Proxy Manager e os próximos da central de aplicações internas).

### Conceitos-chave
| Termo | O que é |
|---|---|
| Ambiente (Environment) | A instância Docker (ou Kubernetes) gerenciada pelo Portainer. Pode ser local ou remota (Edge Agent). |
| Stack | Um conjunto de containers definidos por um arquivo compose, implantado e gerenciado como uma unidade. |
| Endpoint | Outro nome usado para se referir a um ambiente conectado ao Portainer. |
| RBAC | Controle de acesso baseado em papéis: define o que cada usuário ou time pode ver e fazer. |
| Edge Agent | Componente que permite gerenciar hosts remotos sem expor a API do Docker diretamente. |

### Por que usar Portainer
- **Produtividade**: reduz a necessidade de comandos manuais para tarefas do dia a dia.
- **Visibilidade**: painéis de uso de CPU, memória e status dos containers.
- **Padronização**: implantação de stacks a partir de arquivos compose versionados.
- **Controle de acesso**: usuários, times e permissões por ambiente.

## Parte 2: Instalação
### Visão geral

O Portainer foi instalado na mesma máquina virtual do Docker Engine, com sistema operacional Ubuntu, via `docker run` (sem Docker Compose no host).

### Pré-requisitos
- Docker Engine instalado ([01-docker.md](01-docker.md))
- Acesso root ou sudo
- Porta 9443 livre para o primeiro acesso

### Conectando-se ao servidor
```bash
ssh usuario@IP-DO-SERVIDOR
```

### Criar o volume de dados
```bash
docker volume create portainer_data
```

| Volume | Descrição |
|---|---|
| `portainer_data` | Armazena todas as informações persistentes do Portainer, incluindo usuários, senhas, stacks, endpoints Docker gerenciados, configurações de acesso e dados operacionais da interface de administração. Garante que o ambiente seja mantido mesmo após reinicializações ou recriações do container. |

### Instalar o Portainer
```bash
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:2.21.5
```

> Confirme a versão LTS mais recente em https://docs.portainer.io antes de
> instalar. Evite `latest`: uma atualização inesperada pode quebrar a
> compatibilidade sem aviso. A porta 8000 só é necessária para Edge Agents.

### Primeiro acesso
1. Acessar `https://IP-DO-SERVIDOR:9443`
2. Aceitar o certificado autoassinado (temporário, até o proxy reverso estar configurado)
3. Criar o usuário administrador com senha forte, guardada em um gerenciador de senhas
4. Selecionar o ambiente **Get Started** (Docker local)

> Há um tempo limite para a criação do administrador. Se expirar:
> ```bash
> docker restart portainer
> ```

## Parte 3: Comandos

### Gerenciamento do container
```bash
docker ps --filter name=portainer     # Verifica se o Portainer está rodando
docker logs portainer                 # Exibe os logs do Portainer
docker restart portainer              # Reinicia o Portainer
docker stop portainer                 # Para o Portainer
```

### Atualização
```bash
docker stop portainer
docker rm portainer
docker pull portainer/portainer-ce:<nova-versao>
# recriar com o mesmo comando de instalação, alterando a tag
```

O volume `portainer_data` preserva usuários, stacks e configurações durante a atualização.

### Backup do volume
```bash
docker run --rm \
  -v portainer_data:/data \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/portainer-$(date +%F).tar.gz -C /data .
```

### Restore do volume
```bash
docker run --rm \
  -v portainer_data:/data \
  -v $(pwd)/backups:/backup \
  alpine sh -c "cd /data && tar xzf /backup/<arquivo>.tar.gz"
```

## Parte 4: Boas práticas

### Usuários e RBAC
| Perfil | Permissão sugerida |
|---|---|
| Administrador | Somente 1 ou 2 contas nominais |
| Operador | Gerenciar containers e stacks |
| Somente leitura | Consulta e auditoria |

- Uma conta por pessoa, sem compartilhar o admin
- Agrupar usuários em **Teams**
- Restringir o acesso por ambiente
- Guardar a conta de emergência no gerenciador de senhas

### Implantação de stacks
**Stacks → Add stack**

| Método | Quando usar |
|---|---|
| Web editor | Testes e ajustes rápidos |
| Repository | Produção: versionado no Git |
| Upload | Compose local |

> As variáveis do `.env` não são lidas automaticamente pelo Portainer;
> preencha-as em **Environment variables** ao criar a stack.

### Integração com o proxy reverso
1. Implantar o Nginx Proxy Manager como stack (ver [03-nginx-proxy-manager.md](02-nginx-proxy-manager.md))
2. Criar o Proxy Host apontando para `portainer:9443` (esquema HTTPS)
3. Validar o acesso pelo domínio
4. **Hardening:** recriar o container sem publicar a porta 9443, ou restringi-la a `127.0.0.1:9443:9443`

### Segurança
- Nunca usar `latest`: fixar a versão da imagem
- Restringir o acesso ao `docker.sock`, que equivale a root no host
- Fechar a porta 9443 externamente após validar o proxy reverso
- Revisar periodicamente usuários e permissões
