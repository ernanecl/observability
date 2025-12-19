Instalação do Kind (Windows)

Como você está no Windows, existem 3 formas corretas.
Recomendo a Opção A (oficial e limpa).

🅰️ Opção A — Instalação oficial (recomendada)
1️⃣ Baixar o binário

Abra PowerShell e execute:

curl -Lo kind.exe https://kind.sigs.k8s.io/dl/v0.23.0/kind-windows-amd64

2️⃣ Criar pasta para binários (se não existir)
mkdir C:\bin

3️⃣ Mover o arquivo
Move-Item .\kind.exe C:\bin\kind.exe

4️⃣ Adicionar ao PATH
Temporário (sessão atual):
$env:Path += ";C:\bin"

Permanente (recomendado):
setx PATH "$env:PATH;C:\bin"


⚠️ Feche e reabra o terminal depois.


5️⃣ Validar
kind version


✔️ Esperado:

kind v0.23.0

🅱️ Opção B — Chocolatey (se já usa)
choco install kind -y


Validar:

kind version

🅲 Opção C — Scoop (se usa dev tools)
scoop install kind

🔎 Verificação extra (Docker)

Kind depende do Docker rodando.

Confirme:

docker info


Se o Docker Desktop não estiver rodando, o Kind não cria cluster.

✅ Próximo teste (obrigatório)

Depois de instalado:

kind create cluster --name observability


ou com config:

kind create cluster --config observability/kind/kind.yaml






ao executar o comando de adicionar o path, diz que foi sucesso, mas depois que fecho e reabro o terminal não encontra o kind

PS C:\bin> setx PATH "$env:PATH;C:\bin"
   
WARNING: The data being saved is truncated to 1024 characters.

SUCCESS: Specified value was saved.

PS C:\Users\ernan_g2q2z1c> kind version
kind : The term 'kind' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
At line:1 char:1
+ kind version
+ ~~~~
    + CategoryInfo          : ObjectNotFound: (kind:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException

Perfeito, excelente diagnóstico 👍
Esse WARNING é a chave do problema:

WARNING: The data being saved is truncated to 1024 characters.


👉 O setx PATH truncou o PATH do Windows, então o C:\bin não entrou de fato (ou entrou, mas o PATH ficou quebrado).

Vamos corrigir do jeito certo e seguro, sem quebrar mais nada.

✅ SOLUÇÃO CORRETA (sem usar setx)
🔥 O que NÃO fazer

❌ Não use mais:

setx PATH "$env:PATH;C:\bin"


Isso é um problema conhecido do Windows.

🟢 SOLUÇÃO 1 (RECOMENDADA) — Adicionar PATH pelo Windows (GUI)
1️⃣ Abra:

Configurações

Sistema

Sobre

Configurações avançadas do sistema

Variáveis de Ambiente

2️⃣ Em Variáveis do usuário:

Selecione Path

Clique em Editar

Clique em Novo

Adicione:

C:\bin


Clique OK → OK → OK

3️⃣ Feche TODOS os terminais

Abra um novo PowerShell e teste:

kind version


✔️ Agora vai funcionar.



🟢 SOLUÇÃO 2 — Teste rápido (sem mexer no sistema)

Se quiser testar agora, sem reiniciar nada:

$env:Path += ";C:\bin"
kind version


⚠️ Isso funciona apenas na sessão atual.

🟢 SOLUÇÃO 3 — Usar local absoluto (debug)

Só para confirmar que o binário está ok:

C:\bin\kind.exe version


Se isso funcionar → o problema é 100% PATH.

🔎 Verificação rápida do arquivo

Confirme:

dir C:\bin


Deve existir:

kind.exe


🧠 Resumo técnico (importante para DevOps)

setx tem limite de 1024 caracteres

PATH moderno do Windows normalmente tem > 2k caracteres

Resultado: variável corrompida ou incompleta

Sempre use o Editor de Variáveis do Windows para PATH

🚀 Próximo passo (quando kind version funcionar)

Seguimos exatamente assim:

Criar cluster Kind com kind.yaml

Criar namespace observability

Instalar Helm

Deploy do otel-collector como dono total dos sinais

Instalar Tempo, Loki e Mimir

Quando o kind version responder corretamente, me avise que seguimos 🔥