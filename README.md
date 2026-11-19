# Triagem automática de feedbacks de clientes com IA (n8n)

Fluxo de automação que recebe feedbacks de clientes de um app de delivery, classifica cada um com IA e avisa por e-mail quando o caso é urgente.

Projeto de portfólio feito por mim para praticar automação com IA.

![Fluxo no n8n](prints/fluxo.png)

## O problema

Uma empresa de delivery recebe muitos feedbacks por dia e não consegue ler e identificar casos graves(cobrança indevida, pedido que não chegou, alergia), já que eles podem ficar escondidos no meio dos outros. Este fluxo organiza os feedbacks numa planilha e dispara um alerta quando a urgência é alta.

## Como funciona

```
Formulário de feedback
        ↓
Classificar feedback (IA + formato JSON de saída)
        ↓
Organizar campos
        ↓
Salvar na planilha (Google Sheets)
        ↓
Urgência alta? ── sim ──> Enviar alerta por e-mail (Gmail)
```

1. Um **formulário do n8n** recebe o texto do feedback.
2. Um modelo de IA (gpt-5-mini) lê o texto e devolve, em JSON: sentimento, categoria, urgência e um resumo de uma linha.
3. O resultado, junto com o feedback original e a data, é gravado no **Google Sheets**.
4. Se a urgência for **alta**, o **Gmail** envia um e-mail de alerta com o resumo e o feedback original.

![Planilha com os testes](prints/planilha-testes.png)
![E-mail de alerta](prints/email-alerta.png)

## O que a IA devolve

| Campo | Valores possíveis |
|---|---|
| sentimento | positivo, neutro, negativo |
| categoria | entrega, comida, atendimento, preço, app, outros |
| urgencia | baixa, média, alta |
| resumo | texto curto, em uma linha |

As regras completas de classificação estão no prompt do bloco "Classificar feedback", dentro do fluxo.

## Testes e resultados

Montei casos de teste e comparei a classificação da IA com a que eu esperava. Cada mudança no prompt foi testada de novo com os mesmos casos, para ver se algo quebrava.

| Versão | O que mudou | Resultado |
|---|---|---|
| v1 | Prompt inicial | 4 de 7 corretos |
| v2 | Regra "qualquer reclamação = negativo" | Regressão: um elogio com sugestão de melhoria passou a ser classificado como negativo |
| v3 | Regra de sentimento refinada e categoria "outros" | 7 de 7 corretos |
| v4 | Categoria "comida" e regra de erro de preparo | 7 de 7 nos casos originais (sem regressão). 6 casos novos de comida e entrega, com o mesmo resultado nas duas rodadas |

![Resultados na planilha](prints/planilha-testes.png)

## Problemas encontrados e como resolvi

- **A IA inventava categoria quando faltava informação.** "Ok." saía como "app" e "Vcs são péssimos!!!" como "atendimento". Criei a categoria "outros" e escrevi no prompt para não adivinhar o motivo.
- **Só mudar o prompt não bastou.** O "Ok." continuou como "app" até eu incluir "outros" também na lista de valores permitidos do formato JSON de saída. O prompt e o formato precisam estar alinhados.
- **Uma regra nova quebrou um caso que funcionava.** Ao mandar "qualquer reclamação = negativo", um elogio com uma sugestão de filtro virou negativo. Refinei: negativo só quando há um problema concreto. Por isso, a cada mudança, rodo de novo os mesmos casos.
- **Faltava uma categoria central para o negócio.** Percebi que não havia "comida", provavelmente a reclamação mais comum em delivery, e que "a comida é ruim" caía em "outros". Adicionei "comida" e defini a fronteira com "entrega".
- **Casos ambíguos precisam de regra escrita.** Defini, por exemplo: feedback com elogio e reclamação conta como negativo; problemas do entregador, atraso e embalagem são "entrega"; erro de preparo do restaurante (ingrediente errado, alergia) é "comida"; "atendimento" é só o suporte ao cliente.

## Limitações

- O conjunto de testes é pequeno e os casos foram inventados por mim.
- Usei os mesmos casos para ajustar o prompt e para medir o resultado. Por isso o 7 de 7 mostra que os erros conhecidos foram corrigidos, mas não prova o desempenho em feedbacks novos.
- A urgência é um critério subjetivo. Defini as regras no prompt, mas outra equipe poderia querer outras.
- Testei só feedbacks em português e só com o gpt-5-mini. Outros modelos podem se comportar de outra forma.
- O fluxo roda com o formulário de teste do n8n e não está publicado em produção.

## Próximos passos

- Fazer a IA sugerir uma **resposta ao cliente** para cada feedback.
- Ligar a planilha a um **dashboard** (Power BI ou Streamlit) com categorias mais frequentes e sentimento ao longo do tempo.
- Testar com uma base maior de feedbacks.

## Como usar

1. No n8n, importe o arquivo `fluxo-n8n.json` (menu do fluxo → *Import from file*).
2. Crie uma planilha no Google Sheets com estes cabeçalhos na primeira linha: `data`, `feedback`, `sentimento`, `categoria`, `urgencia`, `resumo`.
3. Conecte suas credenciais: Google Sheets, Gmail e o modelo de IA. O fluxo usa o gpt-5-mini, mas dá para trocar por Claude ou Gemini.
4. No bloco **Salvar na planilha**, escolha a sua planilha. No bloco **Enviar alerta por e-mail**, troque `email@exemplo.com` pelo seu e-mail.
5. Clique em *Execute*, abra a *Test URL* do formulário e envie um feedback.

## Ferramentas

n8n · gpt-5-mini · Google Sheets · Gmail

## Autora

Isabela Lemes · [LinkedIn](https://linkedin.com/in/lemesisab) · [GitHub](https://github.com/Lemes-Isab)
