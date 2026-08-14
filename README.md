#  Assistente médico virtual com PydanticAI

Um assistente médico inteligente desenvolvido com **PydanticAI** que utiliza modelos de IA para fornecer orientação médica segura e responsável, integrado com banco de dados de histórico de pacientes.

##  Sobre o Projeto

Este projeto implementa um agente de IA médico que:
-  Avalia sintomas de forma segura e responsável
-  Acessa histórico clínico de pacientes em tempo real
-  Classifica nível de risco (0-10)
-  Identifica necessidade de especialista
-  Detecta situações de emergência
-  Valida entrada de dados com Pydantic
-  Suporta modelos OpenAI/Ollama

##  Aviso de Responsabilidade

**Este projeto é para fins educacionais e demonstrativos apenas.** 

- Não substitui consulta médica profissional
- Sempre recomende ao paciente buscar atendimento médico profissional
- Cumpra todas as regulamentações de privacidade de dados de saúde (LGPD, HIPAA, etc.)

## Recursos Principais

### Agente Inteligente
- Baseado em prompts estruturados e contextualizados
- Acesso a ferramentas dinâmicas (fetch_patient_history)
- Avaliação estruturada de sintomas
- Comunicação clara de incertezas

### Validação de Dados
- Estruturas Pydantic para entrada/saída
- Validação de urgência (escala 1-5)
- Respostas estruturadas com risco e recomendações

### Contexto de Paciente
- Integração com banco de dados médico
- Sistema de protocolo para acesso a histórico
- Personalização de respostas por paciente

### Suporte Multimodelo
- Compatível com OpenAI
- Suporte para Ollama (modelos locais)
- Fácil extensão para outros providers

## Pré-requisitos

- **Python 3.10+**
- pip ou poetry
- OpenAI API key OU Ollama instalado localmente

### Opção 1: OpenAI (Online)
```bash
export OPENAI_API_KEY="sk-..."
```

### Opção 2: Ollama (Local)
```bash
# Instalar Ollama: https://ollama.ai
ollama pull gpt-oss:20b
ollama serve
```

##  Instalação

### 1. Clonar o repositório
```bash
git clone https://github.com/seu-usuario/medical-ai-assistant.git
cd medical-ai-assistant
```

### 2. Criar ambiente virtual
```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou
venv\Scripts\activate  # Windows
```

### 3. Instalar dependências
```bash
pip install -r requirements.txt
```

### Arquivo `requirements.txt`
```
pydantic>=2.0
pydantic-ai>=0.1.0
openai>=1.0.0
python-dotenv>=1.0.0
pytest>=7.0.0
pytest-asyncio>=0.21.0
```

##  Uso Rápido

### Exemplo Básico

```python
from medical_assistant import assist_agent, RequestContext, AssistanceRequest, FakeDB

# Preparar contexto
deps = RequestContext(
    patient_id=42,
    db=FakeDB()
)

# Criar requisição
question = AssistanceRequest(
    symptoms="Dor no peito e falta de ar ao esforço",
    urgency=5
)

# Executar agente
result = await assist_agent.run(
    f"Avalie: {question.model_dump_json()}",
    deps=deps,
)

# Resultado estruturado
print(f"Conselho: {result.output.advice}")
print(f"Nível de Risco: {result.output.risk_level}")
print(f"Referir a Especialista: {result.output.refer_to_specialist}")
```

### Exemplos de Sintomas

```python
# Caso 1: Dor persistente
question = AssistanceRequest(
    symptoms="Dor de cabeça constante há 3 dias",
    urgency=3
)

# Caso 2: Sintomas leves
question = AssistanceRequest(
    symptoms="Tosse seca e leve irritação na garganta",
    urgency=1
)

# Caso 3: Emergência
question = AssistanceRequest(
    symptoms="Dificuldade severa para respirar",
    urgency=5
)
```

##  Estrutura do Projeto

```
medical-ai-assistant/
├── medical_assistant.py      # Código principal do agente
├── database.py               # Interface e implementações de BD
├── models.py                 # Modelos Pydantic (requisição/resposta)
├── config.py                 # Configurações e variáveis de ambiente
├── tests/
│   ├── test_agent.py         # Testes do agente
│   └── test_models.py        # Testes de validação
├── examples/
│   ├── basic_usage.py        # Uso básico
│   ├── with_real_db.py       # Com banco real
│   └── emergency_cases.py    # Casos de emergência
├── requirements.txt
├── .env.example
└── README.md
```

##  Configuração

### Variáveis de Ambiente

Criar arquivo `.env`:

```bash
# OpenAI
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4

# Ollama (local)
OLLAMA_BASE_URL=http://localhost:11434/v1
OLLAMA_MODEL=gpt-oss:20b

# Banco de dados
DATABASE_URL=postgresql://user:pass@localhost/medical_db
DATABASE_TYPE=postgresql  # ou 'mock'

# Segurança
ENABLE_LOGGING=true
LOG_LEVEL=INFO
```

### Alternar entre Modelos

```python
# Com OpenAI
from pydantic_ai.models.openai import OpenAIModel

model = OpenAIModel(model_name='gpt-4')

# Com Ollama
from pydantic_ai.models.openai import OpenAIModel
from pydantic_ai.providers.ollama import OllamaProvider

model = OpenAIModel(
    model_name='gpt-oss:20b',
    provider=OllamaProvider(base_url='http://localhost:11434/v1')
)
```

##  Implementar Banco de Dados Real

### Criar implementação personalizada

```python
from typing import AsyncContextManager
import asyncpg

class PostgresDatabaseImpl:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self.pool = None

    async def connect(self):
        self.pool = await asyncpg.create_pool(self.connection_string)

    async def get_history(self, patient_id: int) -> str:
        async with self.pool.acquire() as conn:
            result = await conn.fetch(
                'SELECT condition FROM patient_history WHERE patient_id = $1',
                patient_id
            )
            return "; ".join([row['condition'] for row in result])

    async def get_patient_name(self, patient_id: int) -> str:
        async with self.pool.acquire() as conn:
            result = await conn.fetchval(
                'SELECT name FROM patients WHERE id = $1',
                patient_id
            )
            return result or "Paciente Desconhecido"

# Uso
db = PostgresDatabaseImpl(os.getenv('DATABASE_URL'))
await db.connect()
deps = RequestContext(patient_id=42, db=db)
```

## Testes

### Executar testes
```bash
pytest tests/ -v
```

### Teste básico
```python
# tests/test_agent.py
import pytest
from medical_assistant import assist_agent, RequestContext, AssistanceRequest, FakeDB

@pytest.mark.asyncio
async def test_basic_symptoms():
    deps = RequestContext(patient_id=1, db=FakeDB())
    question = AssistanceRequest(
        symptoms="Dor no peito",
        urgency=4
    )
    
    result = await assist_agent.run(
        f"Avalie: {question.model_dump_json()}",
        deps=deps,
    )
    
    assert result.output.risk_level > 0
    assert result.output.advice != ""
    assert isinstance(result.output.refer_to_specialist, bool)
```

## Estrutura de Resposta

### Input: `AssistanceRequest`
```json
{
    "symptoms": "Descrição dos sintomas",
    "urgency": 1-5
}
```

### Output: `AssistanceResponse`
```json
{
    "advice": "Recomendação e orientação...",
    "risk_level": 0-10,
    "refer_to_specialist": true/false
}
```

##  Ferramentas Disponíveis

### `fetch_patient_history`
Busca o histórico clínico do paciente no banco.

```python
@assist_agent.tool
async def fetch_patient_history(ctx: RunContext[RequestContext]) -> str:
    """Retorna histórico de condições do paciente."""
```

### Adicionar nova ferramenta

```python
@assist_agent.tool
async def fetch_medications(ctx: RunContext[RequestContext]) -> str:
    """Busca medicações atuais do paciente."""
    return await ctx.deps.db.get_medications(patient_id=ctx.deps.patient_id)
```

## Melhorias Futuras

-  Integração com APIs de farmácias
-  Sistema de feedback e validação de respostas
-  Suporte multilíngue
-  Dashboard de monitoramento
-  Sistema de agendamento de consultas
-  Análise de tendências de sintomas


