# NFS-e HUNT

Aplicativo desktop para baixar, organizar e processar Notas Fiscais de Serviço Eletrônicas (NFS-e) do portal nacional `nfse.gov.br`. Gratuito, sem cadastro, sem coleta de dados, sem nada pra pagar.

![License](https://img.shields.io/badge/license-MIT-blue)
![Platform](https://img.shields.io/badge/windows-10%20%7C%2011-0078D6)
![Python](https://img.shields.io/badge/python-3.10%2B-yellow)
![Status](https://img.shields.io/badge/status-ativo-success)

[Baixar a versão mais recente](../../releases/latest)

---

## Por que esse app existe

Trabalho com escrituração fiscal e baixar NFS-e do portal nacional em volume sempre foi um saco. Clicar nota por nota, esperar download, organizar em pasta, voltar pra próxima. Repete isso pra 30 empresas no fim do mês e o dia já era.

Comecei a automatizar pra mim mesmo em Python com Selenium. Foi crescendo, virou um sistema completo com várias formas de baixar dependendo do caso (login com certificado, lote de empresas cadastradas, controle manual, etc). Achei que ia ser útil pra outras pessoas que enfrentam o mesmo problema, então resolvi disponibilizar gratuitamente.

Continuo desenvolvendo nas horas livres e usando no dia a dia. Se encontrar algum bug ou tiver sugestão, abre uma issue.

---

## O que ele faz

O app tem oito módulos integrados. A ideia é cobrir os diferentes cenários do dia a dia sem precisar de configuração complicada.

### Modos de download

**Modo Manual.** Download interativo nota por nota, com controle total sobre cada parâmetro. Bom quando você precisa de algumas notas específicas e quer acompanhar tudo na tela.

**Modo Manual PDF.** Versão que baixa só os PDFs (DANFS-e), sem os XMLs. Útil quando o cliente quer apenas os comprovantes visuais pra arquivar.

**Modo Híbrido.** Você faz login manualmente (com certificado digital ou senha) e o app cuida do resto: filtros, navegação, downloads, organização em pasta. Funciona bem quando você não quer armazenar credenciais.

**Modo Automático.** Processa em lote todas as empresas que estão cadastradas no sistema. Pode rodar sem supervisão, ideal pra escritório com vários clientes. Configura uma vez e deixa trabalhando.

**Modo Controle Total.** Esse é o mais novo. Abre uma janela flutuante que fica por cima do navegador enquanto você usa o portal normalmente. Tem campos pra definir a competência, o intervalo de datas, o tipo de serviço (tomados ou prestados) e a pasta de saída. Botão de reset desloga o certificado se você precisar trocar de empresa rápido. E tem uma opção que junta tudo num único ZIP no final, o que ajuda bastante quando vai mandar pro contador.

### Ferramentas

**Gerador de Planilhas.** Converte os XMLs baixados em Excel formatado. Tem 57 campos disponíveis (todos os principais do layout nacional) e você escolhe quais quer extrair, em que ordem, com que cabeçalho. Suporta formatação brasileira (vírgula no decimal) por campo. As configurações ficam salvas em JSON local. Reordenar é por arrastar e soltar.

**Gerenciar Empresas.** Cadastro das empresas pra usar no Modo Automático. CNPJ, certificado, credenciais. Tudo guardado localmente e criptografado.

**Configurações.** Painel central pra ajustar pastas padrão, comportamento do navegador, preferências de velocidade e outras opções gerais do sistema.

---

## Instalação

Baixe o instalador da página de [Releases](../../releases/latest):

```
NFSeHunt_Setup_v1.0.0.exe
```

Execute e siga o wizard. Ele instala em `C:\Program Files\NFSeHunt\`, cria atalho na área de trabalho e no menu iniciar.

Na primeira execução o app já abre direto, sem cadastro nem chave de ativação.

---

## Primeiros passos

Pra um teste rápido, recomendo o Modo Controle Total porque é o que dá mais visibilidade do que está acontecendo:

1. Clica no card "Modo Controle Total"
2. Espera o Chrome abrir o portal automaticamente
3. Faz login com o certificado digital
4. Na janela flutuante:
   - Define a competência (formato MM/AAAA, ex: `05/2026`)
   - Clica em "Auto" pra preencher as datas baseado na competência
   - Escolhe o tipo (Tomados ou Prestados)
   - Seleciona a pasta onde quer salvar
5. Clica em "Iniciar Download"

No final aparece um arquivo ZIP único com todos os XMLs e PDFs organizados.

Pra usar o lote automático precisa cadastrar as empresas primeiro em "Gerenciar Empresas".

---

## Requisitos

| Item | Mínimo | Recomendado |
|------|--------|-------------|
| Sistema | Windows 10 (64 bits) | Windows 11 |
| RAM | 4 GB | 8 GB |
| Espaço em disco | 500 MB | 2 GB |
| Navegador | Google Chrome 100+ | Última versão estável |
| Certificado Digital | A1 ou A3 | A1 |

O Chrome é detectado automaticamente. Não precisa configurar nada.

---

## Como funciona por baixo

Pra quem é curioso ou quer entender o que está acontecendo no Chrome durante a automação:

- **Stack**: Python 3.10, Tkinter (interface), Selenium (automação web), openpyxl (geração de Excel), SQLite (armazenamento local), Cryptography (credenciais).
- **Empacotamento**: PyInstaller (onedir) + Inno Setup (instalador). O executável final é standalone, não precisa de Python instalado na máquina do usuário.
- **Downloads**: feitos via URL direta com a chave de acesso de 50 dígitos da nota (endpoint `nfse.gov.br/emissornacional/DPS/ModalCaptcha/DANFSe/{chave}` para PDFs e equivalente pra XMLs). Mais rápido e estável do que abrir o popover e clicar em cada link.
- **Espaçamento**: 2,5 segundos entre cada download pra não sobrecarregar o portal e evitar bloqueio temporário.
- **Cache de downloads** (Modo Controle Total): usa uma pasta temporária do sistema, monitora a chegada dos arquivos via filesystem e empacota tudo em ZIP quando os downloads estabilizam (15 segundos sem novo arquivo).

Não tem comunicação com servidor externo. Tudo é local. Os XMLs e PDFs vão direto do portal pra sua pasta.

---

## O que ele não faz

Pra ser honesto sobre o escopo:

- Não emite notas, só baixa as já emitidas.
- Não integra com sistema contábil. Você precisa importar os XMLs/Excel no seu sistema separadamente.
- Não funciona em Mac ou Linux. Foi desenvolvido pensando em Windows que é o ambiente padrão dos escritórios contábeis brasileiros.
- Não armazena dados em nuvem. Tudo fica no seu computador.
- Não tem agendador interno. Pra rodar automaticamente em horários fixos, integre com o Agendador de Tarefas do Windows.
- Não testa NFS-e de municípios que não estão no portal nacional. Cada município que ainda tem portal próprio fica fora.

---

## Limitações conhecidas

Alguns pontos que vale saber:

- Algumas notas demoram pra completar o download por causa de páginas intermediárias do portal. O Modo Controle Total tem um sistema de cache que aguarda isso resolver, mas em outros modos pode acontecer de uma ou outra nota não baixar na primeira tentativa.
- Janela flutuante do Modo Controle Total tem largura fixa (420 px) pra preservar o layout dos componentes. A altura é ajustável e adapta automaticamente a telas pequenas com scroll vertical.
- O portal nacional eventualmente passa por instabilidade. Quando isso acontece, o app reflete o problema. Nada que dê pra resolver do lado do cliente.

---

## Contribuindo

Se encontrar bug, comportamento estranho ou tiver sugestão de feature:

- **Bug ou problema** abre uma [issue](../../issues/new) descrevendo o que aconteceu, qual versão e qual modo estava usando. Print da tela ajuda muito.
- **Sugestão de feature** também abre uma issue. Não prometo implementar tudo, mas leio todas.
- **Pull request** bem-vindo. Se for mudança grande, prefiro discutir antes numa issue pra evitar retrabalho.

Não tenho tempo de dar suporte individual por email ou WhatsApp. Tudo passa pelas issues do GitHub pra ficar registrado e ajudar outras pessoas.

---

## Atualizações

A versão mais recente está sempre em [Releases](../../releases/latest). Quando tem update grande, posto uma issue marcada como "announcement".

Não tem atualização automática dentro do app. Você precisa baixar o instalador novo e rodar. Configurações e dados são preservados (ficam em `%APPDATA%\NFSeHunt\`).

---

## Licença

MIT. Use, modifique, distribua à vontade. Se ajudar muito, considera deixar uma estrela no repositório, mas não é obrigatório.

Veja [LICENSE](LICENSE) para o texto completo.

---

## Contato

Desenvolvido por Daniel ([OWL Automações](#)) em Belo Horizonte.

Pra dúvidas sobre o projeto, use as issues. Pra outros assuntos, o contato está no perfil do GitHub.

---

Se esse app te ajudou a economizar algumas horas no fim do mês, fica o pedido: deixa uma estrela no repositório. É a única coisa que peço.
