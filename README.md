# Triagem automática de feedbacks com IA

Automação em **n8n** que classifica feedbacks de clientes com IA e separa os casos urgentes dos demais.

▶️ **[Ver demo em vídeo (2 min)](https://drive.google.com/file/d/1s5-bsEcBhvbkLv4A-lRooctcHXn4OTkU/view?usp=sharing)**

## Como funciona

```text
Formulário → IA (Gemini) classifica → regra decide → planilha registra
```

A IA define categoria, sentimento, prioridade e área responsável. Uma regra fixa envia os casos de prioridade alta para a aba *Urgentes* e os demais para a *Fila normal*.

## Resultado

Testei com 30 casos com gabarito: 20 para ajustar o prompt e 10 reservados para o teste final.

- **90%** de acerto na prioridade
- **0** casos urgentes perdidos

**Limitação:** a amostra é pequena e feedbacks vagos ainda geram erros, então em uso real eu manteria revisão humana.

**Ferramentas:** n8n · Google Gemini API (3.5 Flash Lite) · Google Sheets

## Detalhes do processo

<details>
<summary><b>Como o fluxo funciona, passo a passo</b></summary>

1. **Formulário (n8n Form):** o cliente envia o feedback.
2. **Classificação (Gemini):** a IA devolve categoria, sentimento, prioridade, área responsável e um resumo, sempre em JSON.
3. **Conversão:** o texto da IA é transformado em campos e unido ao texto original e à data de envio.
4. **Decisão (If):** uma regra fixa verifica se a prioridade é `alta`.
5. **Registro (Google Sheets):** casos urgentes vão para a aba *Urgentes*; os demais, para *Fila normal*.

**Princípio:** a IA interpreta o texto, mas a decisão final segue uma regra previsível. O texto original é salvo fora da IA, para permitir auditoria.

</details>

<details>
<summary><b>O que o prompt define</b></summary>

- Formato de resposta em JSON, com valores permitidos para cada campo.
- Regras de prioridade aplicadas em ordem, parando na primeira que se aplica.
- Regras para definir a área responsável.
- Proteção contra instruções escritas dentro do feedback (*prompt injection*).

</details>

<details>
<summary><b>Como testei e o que aprendi</b></summary>

Criei **30 casos de teste com gabarito**, incluindo casos ambíguos, sarcasmo, alergia e tentativas de *prompt injection*.

- **20 casos (ajuste):** usados para melhorar o prompt em 3 versões.
- **10 casos (teste final):** reservados e rodados uma única vez, com o prompt congelado.

| Versão | Prioridade | Todos os campos certos | Erros graves* |
|---|---|---|---|
| v1 (ajuste) | 85% | 75% | 0 |
| v2 (ajuste) | 80% | 75% | 1 |
| v3 (ajuste) | 90% | 80% | 0 |
| **v3 (teste final, 10 casos)** | **90%** | **70%** | **0** |

\* Erro grave: caso que deveria ser `alta`, mas a IA classificou com prioridade menor.

**Aprendizados**

- A v2 teve uma regressão: uma regra nova melhorou a área responsável, mas gerou um erro grave. Por isso, cada mudança no prompt foi medida de novo.
- Parte dos erros não era da IA, e sim de regras vagas ou do próprio gabarito.
- Nos casos testados, a IA identificou o sarcasmo e não obedeceu às tentativas de *prompt injection*.
- Ao bater no limite diário da API, troquei para um modelo com cota maior e mantive o mesmo modelo em toda a avaliação.

</details>

<details>
<summary><b>Limitações e próximos passos</b></summary>

**Limitações**

- O teste final teve só 4 casos urgentes. Não dá para afirmar que a automação nunca perderá um caso grave.
- Feedbacks vagos (ex.: "horrível") podem receber uma causa que o cliente não informou.
- Houve um falso alarme no teste final: um caso marcado como urgente sem ser.

**Próximos passos**

- Testar com mais casos e dados reais anonimizados.
- Enviar alertas automáticos para os casos urgentes.
- Tratar respostas da IA fora do formato sem interromper o fluxo.

</details>

---

Feito por **Letícia Murici** como projeto de aprendizado em automação com IA.
