# Rune LanScope

Descobre, identifica e diagnostica os equipamentos da sua rede local: quem está na rede, o que cada um é, quais serviços responderam e o que merece atenção primeiro.

> 📸 _Print do inventário e da topologia_

> ⚠️ Use o Rune LanScope só em redes e equipamentos que você tem autorização para analisar.

## Sobre

Chegar numa rede sem documentação e descobrir o que existe nela costuma ser trabalho de várias ferramentas e muita planilha: um scanner para achar os IPs, outro para as portas, consulta ao DHCP para os nomes, e no fim ainda sobram dezenas de `host-201` sem identificação. O Rune LanScope junta essas fontes num painel só, cruza as evidências para dizer o tipo provável de cada equipamento, guarda o histórico para você ver o que mudou e gera relatórios prontos para a gestão e para a equipe técnica.

Ele responde, na prática:
- Quais equipamentos estão na rede?
- Qual o IP, nome, fabricante e tipo provável de cada um?
- Quais portas e serviços responderam?
- O que deve ser revisado primeiro?
- Quais IPs não responderam e podem ser candidatos a IP fixo?

## Funcionalidades

- **Descoberta combinada**: ping, ARP, DNS reverso e TCP nativos, mais Nmap e NirSoft quando instalados
- **Identificação de interfaces web** sem fazer login: lê título, cabeçalhos e certificado TLS. Reconhece Lenovo XClarity, Dell iDRAC, HPE iLO, Supermicro/IPMI, OpenBMC, IBM IMM, pfSense, TP-Link, FOG Project, Checkmk, Passbolt e Xibo
- **Inventário com tipo provável**, fabricante, portas e as evidências usadas em cada conclusão
- **Prioridade de revisão** e o filtro **Somente ação**, que mostra só o que precisa de atenção
- **Topologia inferida** por sub-rede, com exportação em PNG e PDF
- **Histórico e comparação** entre varreduras: equipamentos novos, os que sumiram e portas novas
- **Cadastro manual** de nome, tipo, IP fixo, responsável e setor, mantido pelo MAC nas próximas varreduras
- **IPs candidatos a livres** por sub-rede
- **Diagnósticos por equipamento**: DNS, ping/TCP, Windows, web e captura passiva com TShark
- **Parecer com IA local** via Ollama, sem enviar o inventário para fora
- **Relatórios**: executivo em HTML, técnico em HTML e técnico em JSON
- **Acesso pela rede** opcional, com token, limite de tentativas e log de auditoria das varreduras
- **App para Windows**, com instalador ou versão portátil

## Como funciona

A cada varredura, o Rune LanScope:

1. Normaliza o alvo informado (IP, CIDR ou intervalo).
2. Descobre os equipamentos com ping, ARP, DNS reverso e conexões TCP do próprio sistema.
3. Aprofunda com o Nmap, se instalado, identificando portas, serviços e produtos.
4. Soma as informações do NirSoft Wireless Network Watcher, se habilitado.
5. Verifica interfaces HTTP/HTTPS e Telnet **sem tentar autenticar**.
6. Junta resultados repetidos e infere o tipo provável de cada equipamento.
7. Mostra inventário, topologia, diagnósticos e relatórios, com o progresso em tempo real.

Os inventários ficam no IndexedDB do navegador por 30 dias, até 25 varreduras.

**Como ler os resultados:**
- **Porta aberta** quer dizer que um serviço respondeu, não que existe vulnerabilidade.
- **Tipo inferido** é uma estimativa baseada em evidências. Sem evidência suficiente, aparece como desconhecido.
- **Prioridade** é uma ordem sugerida para revisão humana, não prova de falha.
- **Sub-rede/VLAN** é o segmento IP observado. Uma VLAN real só se confirma com dados do switch.

## Requisitos

- **Windows** (é o foco do projeto)
- Node.js 22+, só para rodar pelo código-fonte

Opcionais, mas recomendados:

| Ferramenta | Para que serve |
|---|---|
| [Nmap](https://nmap.org/download) | Descoberta mais completa e identificação de serviços |
| [Wireshark](https://www.wireshark.org/download.html) com TShark | Diagnóstico passivo de tráfego (mantenha o TShark marcado na instalação) |
| [Ollama](https://ollama.com/download/windows) | Pareceres com IA local. Sugestão de modelo: `qwen2.5:3b` |
| Utilitários [NirSoft](https://www.nirsoft.net) | Fonte auxiliar de descoberta. Veja [`tools/nirsoft/LEIA-ME.md`](tools/nirsoft/LEIA-ME.md) |

Sem nenhuma delas, o Rune LanScope ainda funciona com ping, ARP, DNS e TCP nativos.

## Como rodar

```powershell
npm install
Copy-Item .env.example .env
npm run dev
```

Abra http://127.0.0.1:3005. Se a porta estiver ocupada, o servidor tenta as próximas e mostra o endereço no terminal.

Para usar a IA: `ollama pull qwen2.5:3b`. O resto funciona sem ela.

Algumas coletas, como a captura passiva e certos dados do Windows, pedem o PowerShell **como Administrador**.

### Acessar de outro computador

Por segurança, o painel só aceita conexões da própria máquina. Para liberar na rede local, edite o `.env`:

```env
ALLOW_LAN_ACCESS=true
API_TOKEN="um-token-com-pelo-menos-24-caracteres"
```

Reinicie e abra `http://IP-DO-COMPUTADOR:3005`. A sessão vale por até 8 horas. Não reaproveite uma senha pessoal como token e nunca exponha a porta na internet.

### Alvos aceitos

```text
192.168.1.0/24
10.1.1.10
10.1.1.1-254
192.168.1.0/24, 192.168.2.0/24
```

Na primeira vez, prefira uma rede `/24`. Redes maiores levam mais tempo e geram mais tráfego.

## Configuração (`.env`)

| Variável | Padrão | Descrição |
|---|---|---|
| `PORT` | `3005` | Porta inicial do servidor |
| `ALLOW_LAN_ACCESS` | `false` | Libera o acesso de outras máquinas da rede |
| `API_TOKEN` | vazio | Token de 24+ caracteres, obrigatório com o acesso pela rede |
| `OLLAMA_URL` | `http://localhost:11434` | Endereço do Ollama |
| `OLLAMA_MODEL` | `qwen2.5:3b` | Modelo dos pareceres |
| `OLLAMA_NUM_CTX` / `OLLAMA_NUM_THREAD` | `2048` / `4` | Contexto e threads do modelo |
| `OLLAMA_KEEP_ALIVE` | `0s` | Quanto tempo o modelo fica carregado depois de responder |
| `DISABLE_LIVE_SCAN` | `false` | Bloqueia varreduras reais |
| `MAX_SCAN_ADDRESSES` | `4096` | Endereços por varredura (até um `/20`) |
| `MAX_SCAN_TARGETS` | `16` | Blocos de alvo por varredura |
| `MAX_CAPTURE_SECONDS` | `20` | Duração máxima da captura passiva |
| `API_RATE_LIMIT` | `180` | Requisições por IP por minuto |
| `AUTH_RATE_LIMIT` | `5` | Tentativas de login por IP a cada 15 minutos |
| `HEAVY_RATE_LIMIT` | `20` | Varreduras, capturas e pareceres por IP a cada 10 minutos |
| `AUDIT_LOG_PATH` | `logs/lanscope-audit.jsonl` | Log de auditoria das varreduras |

O log de auditoria registra horário, origem, escopo, duração e resultado de cada varredura. Tokens, inventários e respostas da IA não são gravados.

## Problemas comuns

**Nenhum equipamento apareceu.** Confira a rede e o alvo, rode o PowerShell como Administrador, veja se o firewall permite ICMP/ARP e ative o Nmap.

**Equipamento aparece como `host-201` ou tipo desconhecido.** O IP respondeu, mas não deu pistas suficientes. Mantenha Nmap e NirSoft ativos, use os diagnósticos de DNS e Windows nos detalhes do equipamento ou faça o cadastro manual, que fica guardado pelo MAC. Celulares e notebooks com MAC aleatório podem mostrar um fabricante que não é o real.

**Nmap ou TShark aparece como indisponível.** Reabra o PowerShell depois de instalar e teste `nmap --version` e `tshark --version`. O Rune LanScope procura nos caminhos padrão (`C:\Program Files\Nmap` e `C:\Program Files\Wireshark`).

**O parecer com IA falhou.** Confira se o Ollama está rodando (`ollama serve`) e se o `OLLAMA_MODEL` aparece em `ollama list`.

**Resultado antigo na tela.** Use o botão de lixeira para limpar o inventário salvo e rode de novo.

## Desenvolvimento

```powershell
npm run lint      # TypeScript
npm test          # testes
npm run build     # build de produção (npm start para rodar)
npm run dist:win  # instalador e versão portátil para Windows
```

Os executáveis do Windows são gerados em `artifacts-v3/`. Se os utilitários NirSoft estiverem em `tools/nirsoft/` na hora do build, eles vão junto no app.

## Limitações atuais

- Foco em IPv4 e Windows
- Um equipamento pode estar ativo e não responder aos testes
- A topologia é inferida e não representa o cabeamento físico
- VLANs reais ainda não são confirmadas por SNMP ou CLI
- O botão SSH só monta e copia o comando: o app não faz login nem guarda credenciais

## Autor

Criado por **Guilherme Dill** · [RUNE Projects](https://www.runeprojects.com.br) · [GitHub](https://github.com/guilhermealceu)

## Licença

[MIT](LICENSE)

<sub>Nmap, Wireshark, NirSoft e as marcas de hardware citadas pertencem aos seus donos. Este projeto é independente e não tem vínculo com eles.</sub>
