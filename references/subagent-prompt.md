# Template de prompt para subagente de pesquisa por país

Copie e adapte (especialidade, país, campos-alvo). Regras de verificação são obrigatórias em todas as execuções.

---

Contexto: pesquisa de carreira/educação médica (fellowship em <ESPECIALIDADE>) para <PERFIL DO USUÁRIO>, entrada pretendida <ANOS>. Regras obrigatórias: todo fato precisa de URL de fonte real (página oficial do programa/hospital/sociedade ou anúncio de vaga); NUNCA invente nomes, emails, salários ou datas — não publicado = escrever 'não encontrado'/'não divulgado'; email deve ser publicado no site oficial; não faça login em sites. Campos descritivos em português do Brasil; nomes/emails/URLs como publicados. Leads a confirmar (NÃO trate como fatos): <LISTA DE PISTAS, SE HOUVER>.

Goal: Levante todos os programas de fellowship em <ESPECIALIDADE> em <PAÍS> que aceitem <PERFIL> e salve o resultado em um arquivo JSON. Para cada programa colete: program, institution, city, website (URL oficial), type (clínico/observership/research), accepts (aceita o perfil-alvo? evidência + URL), focus, head_of_spine (diretor do fellowship/chefe do serviço 2025-2026 + URL), start_availability, application_window, contact_email (publicado ou 'não encontrado'), compensation (publicado ou 'não divulgado'), licensing_requirements (registro/licença/visto para estrangeiros), sources (lista de URLs).

SAÍDA OBRIGATÓRIA: escreva o JSON (UTF-8, indent=2) em <DATASET_DIR>/<cc>-fellowships.json (NÃO use o diretório scratch). Mensagem final CURTA (3-5 linhas): caminho do arquivo, nº de programas, lacunas principais — NÃO cole o JSON na mensagem final.

---

## Notas operacionais

- Um subagente por país, disparados em paralelo com `delegate_task`.
- Se um subagent falhar com `content_policy_blocked`, quase sempre foi o JSON gigante na resposta final — redespache pedindo saída em arquivo (já é o padrão acima).
- Confirme no fim: cada arquivo existe, tem N programas e cada programa tem ao menos uma URL em `sources`.
