# netops-lab

Laboratório de introdução à gestão usando protocolos — **gNMI**, **gNOI**, **NETCONF** e RESTCONF.

O exercício usa um router **Nokia SR Linux** em container, orquestrado pelo **Containerlab**, e explora as interfaces de gestão que substituem a CLI manual e o SNMP em redes modernas.

---

## Objetivos

No final deste laboratório o aluno deve ser capaz de:

- Compreender o papel do **gRPC e Protobuf** como base dos protocolos modernos de gestão
- Usar o **gNMI** para ler configuração e estado (GET), modificar configuração (SET) e subscrever telemetria em tempo real (SUBSCRIBE)
- Distinguir os três modos de subscrição gNMI: `sample`, `on_change` e `once`
- Usar o **gNOI** para executar operações no router (ping, traceroute, ficheiros)
- Usar o **NETCONF** para interagir com o router via XML sobre SSH
- Usar **JSON-RPC** para interagir com o router via HTTP
- Comparar os protocolos em termos de transporte, formato e casos de uso
- Navegar em modelos **YANG** e perceber a estrutura de paths como `/interface[name=mgmt0]/statistics`

Para contexto teórico sobre gRPC, Protobuf e sintaxe `.proto`, consulta os slides disponibilizados pelo docente.

---

## Pré-requisitos

É necessário um terminal **Ubuntu** (ou distribuição Debian-like) com Docker disponível. Consoante o teu sistema operativo:

- **Linux** — tens um terminal Ubuntu nativo. Instala o [Docker Engine](https://docs.docker.com/engine/install/ubuntu/).
- **Windows** — instala o [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) com Ubuntu, e o [Docker Desktop](https://www.docker.com/products/docker-desktop/) com integração WSL2 ativa.
- **macOS (Apple Silicon)** — instala o [OrbStack](https://orbstack.dev) e cria uma máquina Linux com `orb create ubuntu`. Entra com `orb shell`.

A partir daqui, todos os comandos são iguais independentemente do sistema operativo.

---

## Instalação das ferramentas

### 1. Containerlab

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

Verifica:

```bash
containerlab version
```

### 2. gnmic — cliente gNMI

```bash
bash -c "$(curl -sL https://get-gnmic.openconfig.net)"
```

Verifica:

```bash
gnmic version
```

### 3. gnoic — cliente gNOI

```bash
bash -c "$(curl -sL https://get-gnoic.kmrd.dev)"
```

Verifica:

```bash
gnoic version
```

### 4. netconf-console2 — cliente NETCONF

```bash
sudo apt install -y python3 python3-pip
pip3 install netconf-console2 six --break-system-packages
echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc
source ~/.bashrc
```

Verifica:

```bash
netconf-console2 --version
```

---

## Configuração do cliente gNMI

Cria o ficheiro de configuração para não repetir credenciais em cada comando:

```bash
cat > ~/.gnmic.yml << 'EOF'
username: admin
password: NokiaSrl1!
skip-verify: true
encoding: json_ietf
targets:
  clab-netops-lab-srl:57400:
EOF
```

---

## Arrancar o lab

Clona o repositório e lança o lab:

```bash
git clone https://github.com/adcosta/netops-lab 
cd netops-lab
sudo containerlab deploy -t lab.yml
```

Aguarda cerca de 30 segundos até o SR Linux estar operacional. Verifica o estado:

```bash
sudo containerlab inspect -t lab.yml
```

---

## Verificar que tudo está pronto

```bash
gnmic capabilities
```

Deves ver a lista de modelos YANG suportados pelo router. O lab está pronto para os exercícios.

---

## Gestão do lab

```bash
# Ver estado
sudo containerlab inspect -t lab.yml

# Parar e destruir
sudo containerlab destroy -t lab.yml

# Entrar no CLI do router
sudo docker exec -it clab-netops-lab-srl sr_cli

# Ver logs do router
sudo docker logs clab-netops-lab-srl
```

---

## Exercícios

Os exercícios detalhados estão em **[lab-guide.md](lab-guide.md)**, organizados por protocolo:

| Secção | Protocolo | Operações |
|---|---|---|
| 1 | gNMI | Capabilities, GET, SET, SUBSCRIBE |
| 2 | gNOI | Ping, Traceroute, File, Time |
| 3 | NETCONF | Hello, Get-config, Get, Get com filtro |
| 4 | JSON-RPC | Get via HTTP |

---

## Credenciais

| Campo | Valor |
|---|---|
| Utilizador | `admin` |
| Password | `NokiaSrl1!` |
| Target gNMI / gNOI | `clab-netops-lab-srl:57400` |
| Target NETCONF | `clab-netops-lab-srl:830` |
| Target JSON-RPC | `http://clab-netops-lab-srl/jsonrpc` |
