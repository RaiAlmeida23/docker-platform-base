# Nginx Proxy Manager: conceitos, instalação, comandos e boas práticas

## Parte 1: O que é o Nginx Proxy Manager

O Nginx Proxy Manager (NPM) é uma ferramenta baseada em Nginx que fornece uma interface gráfica amigável para gerenciar proxies reversos, certificados SSL e hosts virtuais. Com ele, é possível configurar redirecionamentos de tráfego, proteger sites com HTTPS e gerenciar múltiplos domínios sem precisar editar arquivos de configuração manualmente.

Ele é ideal para ambientes Docker, permitindo criar e gerenciar proxies reversos de forma intuitiva, garantindo que as aplicações estejam acessíveis e seguras na rede local ou na internet. O NPM também suporta automação de certificados Let's Encrypt, simplificando a implantação de conexões seguras. Nesta plataforma, ele é o ponto de entrada HTTPS para o Portainer e para os demais serviços da central de aplicações internas.

### Conceitos-chave
| Termo | O que é |
|---|---|
| Proxy Host | Regra que direciona um domínio (ex.: `portainer.example.com`) para um serviço interno (ex.: `portainer:9443`). |
| Certificado SSL/TLS | Arquivo que garante a conexão HTTPS entre o navegador e o serviço. |
| Let's Encrypt | Autoridade certificadora gratuita, usada para emitir certificados automaticamente. |
| Forward Hostname/Port | Endereço interno (nome do container e porta) para onde o NPM envia o tráfego. |

### Por que usar o NPM
- **Interface gráfica**: cria e gerencia proxies sem editar `nginx.conf` manualmente.
- **Certificados automáticos**: renovação do Let's Encrypt sem intervenção manual.
- **Centralização**: um único ponto de entrada HTTPS para várias aplicações internas.

## Parte 2: Instalação

### Visão geral
O NPM foi instalado como stack via interface do Portainer, na mesma
máquina virtual do Docker Engine, com sistema operacional Ubuntu. O arquivo
compose utilizado está em
[`stacks/nginx-proxy-manager/docker-compose.yml`](../stacks/nginx-proxy-manager/docker-compose.yml)

### Pré-requisitos
- Docker Engine e Portainer instalados ([01-docker.md](01-docker.md), [02-portainer.md](02-portainer.md))
- Portas 80 e 443 livres no host
- Domínio ou DNS interno apontando para o servidor (para emissão de certificados)

### Criar os volumes
Antes de criar a stack, criar os volumes que serão utilizados:

```bash
docker volume create npm_data
docker volume create npm_letsencrypt
```

| Volume | Descrição |
|---|---|
| `npm_data` | Armazena as configurações persistentes do NPM: hosts configurados, usuários, regras de proxy e preferências do painel. Sem este volume, as configurações seriam perdidas ao reiniciar o container. |
| `npm_letsencrypt` | Armazena os certificados SSL/TLS gerados ou importados via Let's Encrypt, garantindo o funcionamento contínuo do HTTPS. |

### Acessar o Portainer
1. Abrir o navegador e acessar `https://IP-DO-SERVIDOR:9443`
2. Fazer login com o usuário administrador

### Criar a stack
1. Ir em **Stacks → Add stack**
2. Preencher o campo **Name** com `nginx-proxy-manager`
3. Colar o conteúdo de
   [`stacks/nginx-proxy-manager/docker-compose.yml`](../stacks/nginx-proxy-manager/docker-compose.yml)
   no **Web editor**, ou usar o método **Repository** apontando para este repositório
4. Preencher a variável `NPM_VERSION` em **Environment variables**
   (valores de exemplo em
   [`.env.example`](../stacks/nginx-proxy-manager/.env.example))
5. Desmarcar a opção **Access control**
6. Clicar em **Deploy the stack**

> O Portainer não lê o `.env` do repositório quando o compose é colado no
> Web editor. Preencha as variáveis diretamente no campo **Environment
> variables** da stack.

### Primeiro acesso
1. Acessar `https://IP-DO-SERVIDOR:81` (ou via túnel SSH, se a porta
   estiver restrita a localhost — ver Parte 4)
2. Entrar com as credenciais padrão de primeiro acesso, documentadas
   publicamente pelo próprio NPM:
   - **email:** `admin@example.com`
   - **senha:** `changeme`
3. **Trocar imediatamente** o e-mail e a senha no primeiro login,
   guardando a nova credencial em um gerenciador de senhas
4. Criar o Proxy Host do Portainer (ver Parte 4)

> Se a porta 81 estiver restrita a `127.0.0.1`, acesse por túnel SSH:
> ```bash
> ssh -L 8081:127.0.0.1:81 usuario@IP-DO-SERVIDOR
> # depois abrir http://localhost:8081
> ```

## Parte 3: Comandos

### Gerenciamento via Portainer
| Ação | Onde fazer |
|---|---|
| Ver logs | Stacks → nginx-proxy-manager → Containers → Logs |
| Reiniciar | Containers → nginx-proxy-manager → Restart |
| Atualizar | Stacks → nginx-proxy-manager → Editor → Update the stack (Pull and redeploy) |

### Backup dos volumes
```bash
docker run --rm \
  -v npm_data:/data \
  -v npm_letsencrypt:/etc/letsencrypt \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/npm-$(date +%F).tar.gz -C / data etc/letsencrypt
```

### Restore dos volumes
```bash
docker run --rm \
  -v npm_data:/data \
  -v npm_letsencrypt:/etc/letsencrypt \
  -v $(pwd)/backups:/backup \
  alpine sh -c "cd / && tar xzf /backup/<arquivo>.tar.gz"
```

## Parte 4: Boas práticas
### Configurar um Proxy Host

Processo geral para publicar qualquer serviço interno através do NPM:

1. No menu lateral do NPM, clicar em **Proxy Hosts**
2. Clicar em **Add Proxy Host**
3. Preencher os campos principais:
   - **Domain Names**: o domínio ou subdomínio configurado no DNS (ex.: `app.seudominio.com`)
   - **Scheme**: `http` ou `https`, de acordo com o serviço interno
   - **Forward Hostname / IP**: nome do container (preferível) ou IP/servidor do serviço interno
   - **Forward Port**: porta do serviço interno (ex.: `3000`)
4. Na aba **SSL**:
   - Marcar **Request a new SSL Certificate**
   - Selecionar **Force SSL** e **HTTP/2 Support**
5. Clicar em **Save**

> O NPM gera automaticamente um certificado válido via Let's Encrypt. Se a
> aplicação rodar apenas localmente (sem domínio público resolvível), será
> necessário usar um certificado SSL autoassinado ou uma CA interna, já que
> o desafio do Let's Encrypt não consegue validar o domínio.

### Segurança
- Painel admin (porta 81) restrito a `127.0.0.1`, acessado por túnel SSH ou VPN
- Trocar as credenciais padrão (`admin@example.com` / `changeme`) imediatamente após o primeiro login
- Fixar a versão da imagem no `.env` da stack, evitando `latest`
- Após validar o Proxy Host do Portainer, remover a publicação direta da porta 9443 no host
- Backup regular de `npm_data` e `npm_letsencrypt` (perder o segundo derruba todos os certificados)
