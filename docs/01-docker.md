# Docker: conceitos, instalação e boas práticas

## Parte 1: O que é o Docker
Docker é uma plataforma que permite empacotar uma aplicação junto com tudo que ela precisa para rodar (bibliotecas, dependências, configurações) em uma unidade chamada container. Esse container roda de forma isolada, mas compartilha o kernel do sistema operacional do host, o que o torna muito mais leve do que uma máquina virtual tradicional.

### Conceitos-chave
| Termo | O que é |
|---|---|
| Imagem | Um "molde" somente leitura com tudo que a aplicação precisa (sistema base, dependências, código). Serve de ponto de partida para criar containers. |
| Container | Uma instância em execução de uma imagem. É isolado, mas leve, e pode ser criado, parado e destruído rapidamente. |
| Dockerfile | Arquivo de texto com as instruções para construir uma imagem. |
| Volume | Espaço de armazenamento persistente, usado para guardar dados que não podem se perder quando o container é recriado. |
| Rede (network) | Camada que permite (ou restringe) a comunicação entre containers. |
| Docker Compose | Ferramenta para definir e subir múltiplos containers relacionados a partir de um único arquivo (`docker-compose.yml`). |
| Docker Engine | O motor que executa e gerencia os containers no host. |

### Por que usar Docker
- Portabilidade: a aplicação roda do mesmo jeito no notebook do desenvolvedor, no servidor de teste e em produção.
- Isolamento: cada aplicação tem suas próprias dependências, sem conflitar com outras no mesmo host.
- Leveza: containers compartilham o kernel do host, então sobem em segundos e consomem menos recursos que VMs.
- Padronização: facilita replicar ambientes e automatizar implantações.

## Parte 2: Instalação


## Parte 3: Boas práticas

