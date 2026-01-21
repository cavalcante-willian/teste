# DFe.NET

**Biblioteca .NET moderna para integração com documentos fiscais eletrônicos brasileiros (DFe)**

Biblioteca completa e robusta para geração, validação, assinatura e transmissão de documentos fiscais eletrônicos junto à SEFAZ (Secretaria da Fazenda), desenvolvida exclusivamente em .NET 10 com foco em multiplataforma, precisão fiscal e conformidade legal.

[![.NET](https://img.shields.io/badge/.NET-10.0-512BD4)](https://dotnet.microsoft.com/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Coverage](https://img.shields.io/badge/coverage-97%25-brightgreen.svg)](docs/coverage.md)
[![Cross-Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey)](docs/compatibility.md)

---

## 📋 Sumário

- [Visão Geral](#-visão-geral)
- [Motivação](#-motivação)
- [Documentos Suportados](#-documentos-suportados)
- [Arquitetura](#-arquitetura)
- [Características Principais](#-características-principais)
- [Princípios Fundamentais](#-princípios-fundamentais)
- [Stack Tecnológico](#-stack-tecnológico)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Instalação](#-instalação)
- [Uso Rápido](#-uso-rápido)
- [Roadmap](#-roadmap)
- [Documentação](#-documentação)
- [Contribuição](#-contribuição)
- [Licença](#-licença)

---

## 🎯 Visão Geral

**DFe.NET** é uma biblioteca de classes .NET projetada para abstrair toda a complexidade técnica e burocrática da integração com os sistemas da SEFAZ para emissão de documentos fiscais eletrônicos no Brasil.

A biblioteca fornece:
- **DTOs fortemente tipados** para todos os layouts oficiais (NFe 4.00/3.10, CTe 4.00/3.00, MDFe 3.00, NFCom 1.00, NFSe 1.02/1.01/1.00)
- **Validação automática** contra schemas XSD oficiais da SEFAZ
- **Assinatura digital** compatível com certificados A1 (.pfx)
- **Comunicação resiliente** com webservices SEFAZ (retry, circuit breaker, timeout)
- **Geração de chaves de acesso** com cálculo de dígito verificador
- **Auditoria fiscal completa** (retenção 7 anos conforme legislação)
- **API fluente e idiomática** em português brasileiro

**Objetivo:** Permitir que desenvolvedores foquem nas regras de negócio dos seus sistemas ERP/gestão, delegando toda complexidade fiscal para a biblioteca.

---

## 💡 Motivação

### Problema

A integração com sistemas fiscais eletrônicos brasileiros é notoriamente complexa:

1. **Múltiplos layouts coexistentes**: NFe possui layouts 3.10 e 4.00 ativos simultaneamente, CTe possui 3.00 e 4.00, empresas precisam suportar ambos
2. **Schemas XSD extensos**: NFe 4.00 possui 588 arquivos XSD interdependentes, com validações cruzadas complexas
3. **Webservices instáveis**: SEFAZ apresenta indisponibilidades frequentes, requerendo estratégias de resiliência robustas
4. **Precisão fiscal crítica**: Erros de arredondamento em valores monetários causam divergências e rejeições pela SEFAZ
5. **Assinatura digital específica**: XML Signature com particularidades (canonicalização, algoritmos específicos)
6. **Auditoria obrigatória**: Legislação exige retenção de XMLs e logs de operações por 7 anos
7. **Multiplataforma**: Sistemas modernos rodam em containers Linux, não apenas Windows

### Solução

**DFe.NET** resolve esses problemas através de:

- **Namespace por versão**: `DFe.NFe.V4` e `DFe.NFe.V3` coexistem pacificamente, sem conflitos de tipo
- **DTOs manuais**: Classes C# mapeadas manualmente contra XSD (não auto-geradas), garantindo controle total e manutenibilidade
- **XSD embedded resources**: Validação contra schemas oficiais sem dependências externas, funcionando offline
- **Polly integration**: Retry com backoff exponencial, circuit breaker e timeout configurados especificamente para SEFAZ
- **Decimal sem arredondamento**: Todos os valores monetários/fiscais usam `decimal` com precisão total (zero arredondamento)
- **System.Security.Cryptography.Xml**: Assinatura digital cross-platform (funciona em Linux containers)
- **Auditoria built-in**: Registro automático de todas operações fiscais com retenção configurável
- **.NET 10 puro**: Zero dependências Windows (WPF, WinForms, System.Drawing framework)

---

## 📄 Documentos Suportados

A biblioteca suporta os seguintes documentos fiscais eletrônicos:

| Documento                                | Modelo | Layouts         | Schema Atual          | Webservice SEFAZ              |
| ---------------------------------------- | ------ | --------------- | --------------------- | ----------------------------- |
| **NF-e** (Nota Fiscal Eletrônica)       | 55     | 4.00, 3.10      | Pacote 010b v1.30     | NFeAutorizacao4 / NFeAutorizacao |
| **NFC-e** (Nota Fiscal Consumidor)      | 65     | 4.00, 3.10      | Pacote 010b v1.30     | NFeAutorizacao4 / NFeAutorizacao |
| **CT-e** (Conhecimento Transporte)      | 57     | 4.00, 3.00      | NT 2025.001           | CTeRecepcao4 / CTeRecepcao      |
| **MDF-e** (Manifesto Documentos Fiscais)| 58     | 3.00            | Layout 3.00           | MDFeRecepcao                    |
| **NFCom** (Nota Fiscal Comunicação)     | 62     | 1.00            | NT 2025.x             | NFComAutorizacao                |
| **NFS-e** (Nota Fiscal Serviços)        | —      | 1.02, 1.01, 1.00| Padrão Nacional v1.02 | Recepcao (Padrão Nacional)      |

### Operações Suportadas por Documento

Cada tipo de documento suporta as seguintes operações:

#### NFe / NFCe
- ✅ Autorização (lote síncrono e assíncrono)
- ✅ Consulta protocolo por chave de acesso
- ✅ Inutilização de numeração
- ✅ Cancelamento (evento 110111)
- ✅ Carta de Correção Eletrônica - CCe (evento 110110)
- ✅ Manifestação do Destinatário (eventos 210200, 210210, 210220, 210240)
- ✅ Consulta cadastro contribuinte
- ✅ Consulta status serviço SEFAZ
- ✅ Download XML autorizado (Distribuição DFe)

#### CTe
- ✅ Autorização
- ✅ Consulta protocolo
- ✅ Inutilização
- ✅ Cancelamento (evento 110111)
- ✅ Carta de Correção (evento 110110)
- ✅ Consulta status serviço

#### MDFe
- ✅ Autorização
- ✅ Consulta protocolo
- ✅ Encerramento (evento 110112)
- ✅ Cancelamento (evento 110111)
- ✅ Inclusão de condutor (evento 110114)
- ✅ Consulta status serviço

#### NFCom / NFSe
- ✅ Autorização
- ✅ Consulta protocolo
- ✅ Cancelamento
- ✅ Substituição (NFSe)
- ✅ Consulta status serviço

### Layouts Legados NÃO Suportados

Por decisão arquitetural, layouts descontinuados pela SEFAZ **não serão implementados**:
- ❌ NFe 2.00 (descontinuado em 2012)
- ❌ CTe 2.00 (descontinuado em 2018)
- ❌ MDFe 1.00 (descontinuado em 2016)

**Razão:** Foco em manutenibilidade e conformidade com schemas ativos. Empresas ainda em layouts legados devem migrar para versões atuais (processo suportado pela SEFAZ).

---

## 🏗️ Arquitetura

### Decisões Arquiteturais Fundamentais

#### 1. Namespace por Versão

Diferentes versões de layouts coexistem através de namespaces versionados:

```csharp
using DFe.NFe.V4.Modelos;        // Layout NFe 4.00
using DFe.NFe.V3.Modelos;        // Layout NFe 3.10
using DFe.CTe.V4.Modelos;        // Layout CTe 4.00

// Tipos não conflitam: V4.NotaFiscalEletronica != V3.NotaFiscalEletronica
```

**Benefício:** Sistemas podem processar NFe 4.00 e 3.10 simultaneamente sem workarounds ou reflection tricks.

#### 2. DTOs Manuais vs Auto-Gerados

Classes C# são **escritas manualmente**, não geradas via `xsd.exe` ou ferramentas similares.

**Razão:**
- **Controle total**: Propriedades têm nomes descritivos em português (`RazaoSocial` ao invés de `xNome`)
- **Validação customizada**: FluentValidation integrado, não apenas XSD
- **Documentação rica**: Cada propriedade possui XML doc comments referenciando campo do manual SEFAZ
- **Manutenibilidade**: Refactorings são simples, não dependem de regeneração

**Trade-off aceito:** Manutenção manual quando SEFAZ atualiza schemas (raro: 1-2x por ano).

#### 3. XSD Embedded Resources

Schemas XSD oficiais da SEFAZ são **embedded resources** no assembly.

**Benefícios:**
- ✅ Validação offline (não requer conexão internet)
- ✅ Versão exata do schema garantida (não há risco de usar schema desatualizado)
- ✅ Deploy simplificado (single DLL, sem arquivos externos)
- ✅ Cross-platform (não depende de caminhos filesystem Windows)

**Localização:** `DFe.NFe/V4/Schemas/*.xsd` compilados como `EmbeddedResource`

#### 4. Separação por Documento

Cada tipo de documento fiscal é um **assembly separado**:

```
DFe.Core.dll           # Abstrações, exceções, validadores base
DFe.NFe.dll            # NFe/NFCe específico
DFe.CTe.dll            # CTe específico
DFe.MDFe.dll           # MDFe específico
DFe.NFCom.dll          # NFCom específico
DFe.NFSe.dll           # NFSe Padrão Nacional
DFe.Infraestrutura.dll # HTTP, XML, Segurança, Auditoria
```

**Benefício:** Consumidores referenciam apenas assemblies necessários (sistema que emite apenas NFe não carrega CTe/MDFe).

### Camadas Arquiteturais

```
┌─────────────────────────────────────────────────────────────┐
│                      APLICAÇÃO CONSUMIDORA                  │
│                    (ERP, Sistema de Vendas)                 │
└────────────────────────────┬────────────────────────────────┘
                             │
                ┌────────────┴─────────────┐
                │   DFe.NFe / CTe / MDFe   │  ← APIs específicas por documento
                │   (Modelos, Validadores, │
                │    Serviços)             │
                └────────────┬─────────────┘
                             │
                ┌────────────┴─────────────┐
                │       DFe.Core           │  ← Abstrações, base classes
                │   (Interfaces, Exceções, │
                │    Validadores Comuns)   │
                └────────────┬─────────────┘
                             │
                ┌────────────┴─────────────┐
                │  DFe.Infraestrutura      │  ← Cross-cutting concerns
                │  (HTTP, XML, Segurança,  │
                │   Auditoria, Resiliência)│
                └────────────┬─────────────┘
                             │
            ┌────────────────┴──────────────────┐
            │         SEFAZ Webservices         │
            │  (NFeAutorizacao4, CTeRecepcao4)  │
            └───────────────────────────────────┘
```

### Padrões de Design Utilizados

- **Repository Pattern**: Abstração para persistência de auditoria
- **Factory Pattern**: Criação de clientes HTTP configurados por UF
- **Strategy Pattern**: Algoritmos de validação de IE por UF
- **Builder Pattern**: Construção fluente de XMLs complexos
- **Chain of Responsibility**: Pipeline de validações (FluentValidation → XSD → Regras Negócio)

---

## ⚡ Características Principais

### 1. Cross-Platform Absoluto

Roda nativamente em:
- ✅ Windows 10/11 (x64, ARM64)
- ✅ Linux (Ubuntu 20.04+, Debian 11+, RHEL 8+)
- ✅ macOS 12+ (Intel e Apple Silicon)
- ✅ Docker containers (Alpine, Debian, Ubuntu base images)

**Zero dependências Windows:**
- Sem `System.Drawing` (GDI+)
- Sem `System.Windows.Forms`
- Sem APIs Win32 via P/Invoke
- Certificados A1 via `System.Security.Cryptography` (cross-platform)

### 2. Precisão Fiscal com Decimal

Todos os valores monetários e fiscais usam **`decimal`** (nunca `double` ou `float`):

```csharp
public class Produto
{
    public decimal ValorUnitario { get; set; }    // TDec_1302 (13 dígitos, 2 decimais)
    public decimal Quantidade { get; set; }       // TDec_1104 (11 dígitos, 4 decimais)
    
    public decimal CalcularValorTotal()
    {
        // Multiplicação direta, SEM arredondamento
        return ValorUnitario * Quantidade;
    }
}
```

**Razão:** SEFAZ valida valores com precisão total. Qualquer divergência causada por arredondamento resulta em **Rejeição 215** (falha no schema XML).

### 3. Timezone UTC-3 (Horário de Brasília)

Datas são armazenadas em **UTC** internamente, mas sempre convertidas para **UTC-3** ao serializar XML para SEFAZ:

```csharp
public class TimeZoneConfiguracao
{
    public static readonly TimeZoneInfo TimeZoneBrasil = 
        TimeZoneInfo.FindSystemTimeZoneById("E. South America Standard Time");
    
    public static DateTime AgoraBrasil() => 
        TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, TimeZoneBrasil);
}
```

**Benefício:** Compatibilidade global (sistema pode rodar em servidor europeu UTC+1 e emitir NFe com timestamp correto UTC-3).

### 4. Validação em Duas Camadas

**Camada 1 - FluentValidation** (regras negócio):
```csharp
public class ValidadorEmitente : AbstractValidator<Emitente>
{
    public ValidadorEmitente()
    {
        RuleFor(e => e.Cnpj)
            .NotEmpty()
            .Length(14)
            .Must(CnpjValido).WithMessage("CNPJ com dígito verificador inválido");
            
        RuleFor(e => e.InscricaoEstadual)
            .Must((emitente, ie) => ValidarIEPorUF(ie, emitente.Endereco.UF))
            .WithMessage("IE inválida para UF informada");
    }
}
```

**Camada 2 - XSD Schema** (conformidade SEFAZ):
```csharp
ValidationResult resultado = validadorXsd.Validar(xml, "nfe_v4.00.xsd");
if (!resultado.IsValid)
{
    throw new SchemaValidationException(
        $"XML não conforme: {string.Join(", ", resultado.Errors)}"
    );
}
```

### 5. Resiliência com Polly

Políticas de retry, circuit breaker e timeout específicas para SEFAZ:

```csharp
public static class PoliticasPolly
{
    // Retry: 3 tentativas com backoff exponencial (2s, 4s, 8s)
    public static IAsyncPolicy<HttpResponseMessage> RetryPolicy = 
        Policy
            .HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: tentativa => TimeSpan.FromSeconds(Math.Pow(2, tentativa))
            );
    
    // Circuit Breaker: abre após 5 falhas, aguarda 30s
    public static IAsyncPolicy<HttpResponseMessage> CircuitBreakerPolicy = 
        Policy
            .HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .CircuitBreakerAsync(
                handledEventsAllowedBeforeBreaking: 5,
                durationOfBreak: TimeSpan.FromSeconds(30)
            );
    
    // Timeout: 30 segundos por requisição
    public static IAsyncPolicy<HttpResponseMessage> TimeoutPolicy = 
        Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(30));
}
```

### 6. Auditoria Fiscal Automática

Registro de **todas** operações fiscais conforme exigência legal (retenção 7 anos):

```csharp
public interface IAuditoriaService
{
    Task RegistrarAutorizacaoAsync(
        string chaveAcesso,
        string xmlEnviado,
        string? xmlRecebido,
        bool sucesso,
        string? mensagemErro,
        CancellationToken ct);
}
```

**Armazenamento:**
- XML enviado (requisição completa SOAP)
- XML recebido (resposta SEFAZ com protocolo)
- Timestamp UTC-3
- Usuário responsável
- Status (sucesso/falha/rejeição)
- Código de erro SEFAZ (quando aplicável)

### 7. API Fluente em Português

```csharp
// Criar NFe
var nfe = new NotaFiscalEletronica
{
    Emitente = new Emitente
    {
        Cnpj = "12345678000195",
        RazaoSocial = "EMPRESA TESTE LTDA",
        InscricaoEstadual = "123456789012"
    },
    Destinatario = new Destinatario
    {
        Cnpj = "98765432000123",
        RazaoSocial = "CLIENTE EXEMPLO LTDA"
    },
    Itens = new List<ItemNFe>
    {
        new ItemNFe
        {
            Produto = new Produto
            {
                CodigoProduto = "PROD001",
                Descricao = "Notebook Dell Inspiron",
                Ncm = "84713000",
                ValorUnitario = 3500.00m,
                Quantidade = 2.0000m
            }
        }
    }
};

// Validar
var validador = new ValidadorNotaFiscalEletronica();
var resultadoValidacao = await validador.ValidateAsync(nfe);

if (!resultadoValidacao.IsValid)
{
    throw new ValidationException("NFe inválida", resultadoValidacao.Errors);
}

// Assinar
var certificado = new CertificadoDigital("caminho/certificado.pfx", "senha");
var xmlAssinado = assinadorDigital.Assinar(nfe.ToXml(), certificado);

// Autorizar
var servicoAutorizacao = new ServicoAutorizacaoNFe(httpClientFactory, logger);
var protocolo = await servicoAutorizacao.AutorizarAsync(nfe, certificado, ct);

Console.WriteLine($"NFe autorizada! Protocolo: {protocolo.Numero}");
```

---

## 🎯 Princípios Fundamentais

### 1. 100% Português

**TODO** código, comentários, documentação, commits e mensagens de erro em português brasileiro.

**Razão:** 
- Domínio fiscal é 100% brasileiro (SEFAZ, DANFE, ICMS, IPI, PIS, COFINS)
- Termos técnicos são em português nas especificações oficiais
- Reduz impedância entre documentação SEFAZ e código

**Exceção:** Palavras-chave C#, APIs .NET, bibliotecas externas (imutáveis).

### 2. XSD Como Única Fonte de Verdade

Schemas XSD oficiais publicados pela SEFAZ são a **referência absoluta**:
- DTOs mapeiam exatamente estruturas XSD (tags, tipos, obrigatoriedade)
- Validação contra XSD é **obrigatória** antes de enviar para SEFAZ
- Atualizações de schema refletem em atualização de DTOs

**Exceção:** Validações de negócio adicionais (ex: CNPJ válido, IE por UF) não presentes no XSD.

### 3. Zero Arredondamento

**NUNCA** arredondar valores monetários ou fiscais em cálculos:
- Usar `decimal` com precisão total
- Proibido `Math.Round()`, `double`, `float`
- Comparações exatas com `==` (sem tolerância)

**Razão:** Divergências causam rejeições SEFAZ e multas fiscais.

### 4. Cross-Platform Não Negociável

Biblioteca **DEVE** funcionar em Windows, Linux, macOS e containers:
- Zero APIs Windows-specific
- Caminhos de arquivo via `Path.Combine()` e `Environment.SpecialFolder`
- Certificados A1 (.pfx) apenas (A3/SmartCard não funciona em containers)

**Teste obrigatório:** CI/CD roda em Windows, Ubuntu e macOS.

### 5. Documentação XML Completa

**TODOS** membros públicos (classes, métodos, propriedades) possuem XML doc comments:
- Tag `<summary>` com descrição em português
- Tag `<param>` para cada parâmetro
- Tag `<returns>` para retornos
- Tag `<exception>` para exceções lançadas
- Referência ao campo SEFAZ quando aplicável (ex: "C02 - CNPJ do emitente")

### 6. Cobertura de Testes 97%

Meta de cobertura **não negociável**:
- Serviços: 97%
- Validadores: 97%
- Core (ChaveAcesso, Calculadoras): 97%
- DTOs: não requer testes (apenas propriedades)

**Ferramentas:** xUnit, FluentAssertions, Moq, Coverlet.

### 7. .NET 10 Exclusivo

Biblioteca compila **apenas** para `net10.0`:
- Sem multi-targeting (.NET Framework, .NET Standard, versões antigas)
- Usa recursos modernos (primary constructors, file-scoped namespaces, required members)
- Nullable reference types habilitado

**Razão:** Simplicidade, manutenibilidade, adoção de features modernas.

---

## 🛠️ Stack Tecnológico

### Runtime
- **.NET 10.0** (LTS previsto para novembro 2025)
- **C# 13**

### Bibliotecas Core
- **System.Security.Cryptography.Xml**: Assinatura digital XML
- **System.Xml.Serialization**: Serialização/deserialização XML
- **System.Net.Http**: Comunicação SOAP com SEFAZ

### Validação
- **FluentValidation 11.x**: Validações de negócio
- **System.Xml.Schema**: Validação XSD

### Resiliência
- **Polly 8.x**: Retry, circuit breaker, timeout

### Logging
- **Microsoft.Extensions.Logging**: Abstração logging
- **Serilog** (consumidor escolhe): Logging estruturado

### Testes
- **xUnit 2.x**: Framework de testes
- **FluentAssertions 6.x**: Assertions expressivas
- **Moq 4.x**: Mocking
- **Coverlet**: Cobertura de código

### Ferramentas
- **EditorConfig**: Formatação consistente
- **Roslyn Analyzers**: Análise estática
- **GitHub Actions**: CI/CD

---

## 📂 Estrutura do Projeto

```
DFe.NET/
├── src/
│   ├── DFe.Core/                       # Componentes compartilhados
│   │   ├── Abstratos/                  # Interfaces (IServicoSefaz, IValidadorXsd)
│   │   ├── Excecoes/                   # Hierarquia de exceções customizadas
│   │   ├── Modelos/                    # Value objects (ChaveAcesso, Protocolo)
│   │   ├── Validadores/                # Validadores base (CNPJ, CPF, IE)
│   │   └── Configuracao/               # TimeZone, constantes
│   │
│   ├── DFe.NFe/                        # NF-e / NFC-e
│   │   ├── V4/                         # Layout 4.00
│   │   │   ├── Modelos/                # NotaFiscalEletronica, Emitente, Produto...
│   │   │   ├── Validadores/            # FluentValidation validators
│   │   │   ├── Servicos/               # ServicoAutorizacaoNFe, ServicoConsultaNFe...
│   │   │   ├── Enumeracoes/            # ModeloDocumento, TipoEmissao, TipoOperacao
│   │   │   └── Schemas/                # XSDs embedded (nfe_v4.00.xsd)
│   │   └── V3/                         # Layout 3.10 (estrutura similar)
│   │
│   ├── DFe.CTe/                        # CT-e
│   │   ├── V4/                         # Layout 4.00
│   │   └── V3/                         # Layout 3.00
│   │
│   ├── DFe.MDFe/                       # MDF-e
│   │   └── V3/                         # Layout 3.00
│   │
│   ├── DFe.NFCom/                      # NFCom
│   │   └── V1/                         # Layout 1.00
│   │
│   ├── DFe.NFSe/                       # NFS-e Padrão Nacional
│   │   ├── V1_02/                      # Layout 1.02
│   │   ├── V1_01/                      # Layout 1.01
│   │   └── V1_00/                      # Layout 1.00
│   │
│   └── DFe.Infraestrutura/             # Cross-cutting concerns
│       ├── Http/                       # ClienteHttpSefaz, ManipuladorMensagemSoap
│       ├── Xml/                        # SerializadorXml, DeserializadorXml
│       ├── Seguranca/                  # AssinadorDigital, CertificadoDigital
│       ├── Validacao/                  # ValidadorXsd, ProvedorSchemaEmbutido
│       ├── Auditoria/                  # ServicoAuditoria, logs fiscais
│       └── Resiliencia/                # PoliticasPolly (retry, circuit breaker)
│
├── tests/
│   ├── DFe.Core.Testes/
│   ├── DFe.NFe.Testes/
│   ├── DFe.CTe.Testes/
│   └── DFe.Integracao.Testes/          # Testes integração (mock SEFAZ)
│
├── samples/
│   ├── NFe.AutorizacaoSimples/         # Exemplo básico autorização NFe
│   ├── NFe.Cancelamento/               # Exemplo cancelamento
│   └── CTe.AutorizacaoCompleta/        # Exemplo completo CTe
│
├── docs/
│   ├── PADRONIZACAO.md                 # Padrões de código (2766 linhas)
│   ├── ARQUITETURA.md                  # Decisões arquiteturais detalhadas
│   ├── MIGRACAO_LAYOUTS.md             # Guia migração NFe 3.10 → 4.00
│   └── SCHEMAS_SEFAZ.md                # Documentação schemas por versão
│
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                      # Build, test, coverage
│   │   └── release.yml                 # Publicação NuGet
│   ├── ISSUE_TEMPLATE/
│   └── PULL_REQUEST_TEMPLATE.md
│
├── .editorconfig                       # Formatação (Allman, 4 espaços)
├── .gitignore
├── Directory.Build.props               # Propriedades MSBuild compartilhadas
├── DFe.NET.sln
├── LICENSE
├── README.md
└── CHANGELOG.md
```

---

## 📦 Instalação

### Via NuGet (quando publicado)

```bash
# NFe / NFCe
dotnet add package DFe.NFe

# CTe
dotnet add package DFe.CTe

# MDFe
dotnet add package DFe.MDFe

# NFCom
dotnet add package DFe.NFCom

# NFSe
dotnet add package DFe.NFSe

# Infraestrutura (instalado automaticamente como dependência)
dotnet add package DFe.Infraestrutura
```

### Via Source (desenvolvimento)

```bash
git clone https://github.com/seu-usuario/DFe.NET.git
cd DFe.NET
dotnet restore
dotnet build
dotnet test
```

### Requisitos

- **.NET 10 SDK** instalado
- **Certificado Digital A1** (.pfx) para assinatura
- **Acesso internet** para comunicação com SEFAZ (portas 443 HTTPS)

---

## 🚀 Uso Rápido

### Exemplo 1: Autorizar NFe Básica

```csharp
using DFe.Core.Modelos;
using DFe.NFe.V4.Modelos;
using DFe.NFe.V4.Servicos;
using DFe.Infraestrutura.Seguranca;

// 1. Criar NFe
var nfe = new NotaFiscalEletronica
{
    Identificacao = new Identificacao
    {
        UF = "SP",
        NaturezaOperacao = "Venda de mercadoria",
        Modelo = ModeloDocumento.NFe,
        Serie = 1,
        Numero = 123456,
        TipoOperacao = TipoOperacao.Saida,
        TipoEmissao = TipoEmissao.Normal,
        Ambiente = AmbienteSefaz.Homologacao
    },
    Emitente = new Emitente
    {
        Cnpj = "12345678000195",
        RazaoSocial = "EMPRESA TESTE LTDA",
        InscricaoEstadual = "123456789012",
        RegimeTributario = RegimeTributario.SimplesNacional,
        Endereco = new EnderecoEmitente
        {
            Logradouro = "Rua Teste",
            Numero = "123",
            Bairro = "Centro",
            CodigoMunicipio = "3550308",
            NomeMunicipio = "São Paulo",
            UF = "SP",
            Cep = "01001000"
        }
    },
    Destinatario = new Destinatario
    {
        Cnpj = "98765432000123",
        RazaoSocial = "CLIENTE EXEMPLO LTDA",
        Endereco = new EnderecoDestinatario
        {
            Logradouro = "Avenida Principal",
            Numero = "456",
            Bairro = "Comercial",
            CodigoMunicipio = "3550308",
            NomeMunicipio = "São Paulo",
            UF = "SP",
            Cep = "01310000"
        }
    },
    Itens = new List<ItemNFe>
    {
        new ItemNFe
        {
            Produto = new Produto
            {
                CodigoProduto = "PROD001",
                Descricao = "Notebook Dell Inspiron 15",
                Ncm = "84713000",
                Cfop = "5102",
                UnidadeComercial = "UN",
                QuantidadeComercial = 2.0000m,
                ValorUnitario = 3500.00m,
                ValorTotal = 7000.00m
            },
            Imposto = new Imposto
            {
                Icms = new Icms00
                {
                    Origem = OrigemMercadoria.Nacional,
                    Cst = "00",
                    BaseCalculo = 7000.00m,
                    Aliquota = 18.00m,
                    Valor = 1260.00m
                }
            }
        }
    },
    Total = new Total
    {
        IcmsTotais = new IcmsTotais
        {
            BaseCalculoIcms = 7000.00m,
            ValorIcms = 1260.00m,
            ValorTotalProdutos = 7000.00m,
            ValorTotalNFe = 7000.00m
        }
    }
};

// 2. Carregar certificado digital
var certificado = new CertificadoDigital(
    caminhoArquivo: @"C:\Certificados\empresa.pfx",
    senha: "senha123"
);

// 3. Criar serviço de autorização
var servicoAutorizacao = new ServicoAutorizacaoNFe(
    httpClientFactory,
    logger,
    assinadorDigital,
    validadorXsd,
    provedorEnderecosSefaz
);

// 4. Autorizar
try
{
    Protocolo protocolo = await servicoAutorizacao.AutorizarAsync(
        nfe, 
        certificado, 
        cancellationToken
    );
    
    Console.WriteLine($"✅ NFe autorizada com sucesso!");
    Console.WriteLine($"Protocolo: {protocolo.Numero}");
    Console.WriteLine($"Data: {protocolo.DataRecebimento:dd/MM/yyyy HH:mm:ss}");
}
catch (ValidationException ex)
{
    Console.WriteLine($"❌ Erro de validação:");
    foreach (var erro in ex.Errors)
    {
        Console.WriteLine($"  - {erro}");
    }
}
catch (SchemaValidationException ex)
{
    Console.WriteLine($"❌ XML não conforme com schema SEFAZ:");
    foreach (var erro in ex.Errors)
    {
        Console.WriteLine($"  - {erro}");
    }
}
catch (SefazCommunicationException ex)
{
    Console.WriteLine($"❌ Falha na comunicação com SEFAZ: {ex.Message}");
}
```

### Exemplo 2: Consultar NFe por Chave de Acesso

```csharp
var servicoConsulta = new ServicoConsultaNFe(httpClientFactory, logger);

string chaveAcesso = "35240612345678901234550010000123451234567890";

Protocolo protocolo = await servicoConsulta.ConsultarAsync(
    chaveAcesso,
    certificado,
    cancellationToken
);

Console.WriteLine($"Status: {protocolo.Status}");
Console.WriteLine($"Motivo: {protocolo.Motivo}");
```

### Exemplo 3: Cancelar NFe

```csharp
var servicoCancelamento = new ServicoCancelamentoNFe(httpClientFactory, logger);

string chaveAcesso = "35240612345678901234550010000123451234567890";
string protocolo = "135240000123456";
string justificativa = "Erro no valor do produto - valor correto R$ 3.500,00";

bool cancelado = await servicoCancelamento.CancelarAsync(
    chaveAcesso,
    protocolo,
    justificativa,
    certificado,
    cancellationToken
);

if (cancelado)
{
    Console.WriteLine("✅ NFe cancelada com sucesso!");
}
```

**Mais exemplos:** Consulte a pasta `samples/` no repositório.

---

## 🗺️ Roadmap

### Fase 1 - Fundação (Q1 2026) ✅ Em Progresso
- [x] Estrutura base do projeto
- [x] Documentação de padrões (PADRONIZACAO.md)
- [x] Pipeline CI/CD (GitHub Actions)
- [ ] DFe.Core completo (abstrações, exceções, validadores base)
- [ ] DFe.Infraestrutura (HTTP, XML, segurança, auditoria, resiliência)

### Fase 2 - NFe V4 (Q2 2026)
- [ ] Modelos completos NFe 4.00 (A01-Z13)
- [ ] Validadores FluentValidation + XSD
- [ ] Serviços: Autorização, Consulta, Inutilização, Cancelamento, CCe
- [ ] Testes unitários (cobertura 97%)
- [ ] Testes integração (mock SEFAZ)
- [ ] Publicação NuGet (DFe.NFe v1.0.0-preview)

### Fase 3 - CTe V4 (Q3 2026)
- [ ] Modelos CTe 4.00
- [ ] Validadores CTe
- [ ] Serviços CTe (autorização, consulta, eventos)
- [ ] Testes CTe
- [ ] Publicação NuGet (DFe.CTe v1.0.0-preview)

### Fase 4 - MDFe V3 (Q4 2026)
- [ ] Modelos MDFe 3.00
- [ ] Validadores MDFe
- [ ] Serviços MDFe (autorização, encerramento, eventos)
- [ ] Testes MDFe
- [ ] Publicação NuGet (DFe.MDFe v1.0.0-preview)

### Fase 5 - NFCom e NFSe (Q1 2027)
- [ ] Modelos NFCom 1.00
- [ ] Modelos NFSe (1.02, 1.01, 1.00)
- [ ] Serviços NFCom e NFSe
- [ ] Testes
- [ ] Publicação NuGet

### Fase 6 - Layouts Legados (Q2 2027)
- [ ] NFe 3.10 (coexistência com 4.00)
- [ ] CTe 3.00 (coexistência com 4.00)
- [ ] Publicação versões estáveis (v1.0.0)

### Fase 7 - Features Avançadas (Q3-Q4 2027)
- [ ] DANFE/DACTE/DAMDFE (geração PDF cross-platform com QuestPDF)
- [ ] Contingência offline (FS-DA, EPEC, SVC)
- [ ] Integração com bancos de dados (Entity Framework templates)
- [ ] CLI para operações via terminal
- [ ] Suporte certificados A3 (via PKCS#11 cross-platform)

---

## 📚 Documentação

### Documentos Principais

- **[README.md](README.md)** (este arquivo): Visão geral e quickstart
- **[PADRONIZACAO.md](docs/PADRONIZACAO.md)**: Padrões obrigatórios de código (2766 linhas)
- **[ARQUITETURA.md](docs/ARQUITETURA.md)**: Decisões arquiteturais detalhadas
- **[SCHEMAS_SEFAZ.md](docs/SCHEMAS_SEFAZ.md)**: Documentação schemas XSD por versão
- **[CHANGELOG.md](CHANGELOG.md)**: Histórico de versões

### Guias Específicos

- **[Migração NFe 3.10 → 4.00](docs/MIGRACAO_NFE_3_PARA_4.md)**: Diferenças entre layouts
- **[Validação IE por UF](docs/VALIDACAO_IE.md)**: Algoritmos por estado
- **[Contingência SEFAZ](docs/CONTINGENCIA.md)**: Modos offline (futuro)
- **[Certificados Digitais](docs/CERTIFICADOS.md)**: A1 vs A3, obtenção, renovação

### API Reference

Documentação completa da API será gerada via **DocFX** e publicada em:
- https://seu-usuario.github.io/DFe.NET/api/

### Wiki

Wiki do repositório conterá:
- FAQs
- Troubleshooting comum (rejeições SEFAZ)
- Exemplos práticos por cenário
- Contribuições da comunidade

---

## 🤝 Contribuição

Contribuições são bem-vindas! Este é um projeto open-source comunitário.

### Como Contribuir

1. **Fork** o repositório
2. **Clone** seu fork: `git clone https://github.com/seu-usuario/DFe.NET.git`
3. **Crie um branch** para sua feature: `git checkout -b feat/minha-feature`
4. **Implemente** seguindo **rigorosamente** [PADRONIZACAO.md](docs/PADRONIZACAO.md)
5. **Teste** com cobertura 97%: `dotnet test`
6. **Commit** seguindo Conventional Commits PT: `git commit -m "feat: adiciona validação XYZ"`
7. **Push**: `git push origin feat/minha-feature`
8. **Abra Pull Request** no repositório principal

### Checklist PR

Seu Pull Request **DEVE** atender:
- [ ] Código 100% em português
- [ ] Documentação XML completa (summary, param, returns, exception)
- [ ] Testes xUnit com cobertura ≥97%
- [ ] Zero warnings do compilador
- [ ] Zero violations Roslyn Analyzers
- [ ] Pipeline CI/CD passando (Windows + Linux + macOS)
- [ ] Commit messages em Conventional Commits PT
- [ ] Sem breaking changes não documentados

**Leia atentamente:** [PADRONIZACAO.md](docs/PADRONIZACAO.md) antes de contribuir.

### Áreas Prioritárias para Contribuição

- **Validação IE por UF**: Implementar algoritmos de validação de Inscrição Estadual para estados ainda não cobertos
- **Testes de integração**: Mocks de respostas SEFAZ para diferentes cenários (aprovação, rejeição, contingência)
- **Exemplos práticos**: Samples de uso para cenários reais (venda, devolução, remessa)
- **Documentação**: Tradução de manuais SEFAZ para português claro
- **Performance**: Benchmarks e otimizações de validação XSD

---

## 📜 Licença

Este projeto está licenciado sob **MIT License** - veja o arquivo [LICENSE](LICENSE) para detalhes.

**Resumo MIT:**
- ✅ Uso comercial permitido
- ✅ Modificação permitida
- ✅ Distribuição permitida
- ✅ Uso privado permitido
- ❌ Nenhuma garantia
- ❌ Responsabilidade sobre uso

**Schemas XSD SEFAZ:** Propriedade do Governo Federal Brasileiro, distribuídos publicamente via [Portal Nacional da NFe](http://www.nfe.fazenda.gov.br/).

---

## 🙏 Agradecimentos

- **SEFAZ** (Secretaria da Fazenda): Por especificações públicas e schemas XSD
- **Comunidade .NET Brasil**: Por feedback e sugestões
- **Contribuidores**: Veja [CONTRIBUTORS.md](CONTRIBUTORS.md)

---

## 📞 Suporte

### Issues GitHub
Reporte bugs, solicite features ou tire dúvidas: [GitHub Issues](https://github.com/seu-usuario/DFe.NET/issues)

### Discussões
Participe de discussões: [GitHub Discussions](https://github.com/seu-usuario/DFe.NET/discussions)

### Contato
- **Email**: seu-email@exemplo.com
- **Twitter**: @seu-usuario

---

## ⚠️ Disclaimer

**DFe.NET** é uma biblioteca de software que auxilia desenvolvedores a integrar com sistemas SEFAZ. A responsabilidade pela conformidade fiscal, valores corretos, emissão dentro dos prazos legais e demais obrigações tributárias é **exclusivamente** do contribuinte/empresa usuária.

Os autores e contribuidores **não se responsabilizam** por:
- Multas fiscais decorrentes de uso incorreto da biblioteca
- Rejeições SEFAZ causadas por dados inválidos fornecidos pela aplicação consumidora
- Indisponibilidade dos webservices SEFAZ
- Perdas financeiras ou danos causados pelo uso da biblioteca

**Use por sua conta e risco. Consulte sempre um contador e a legislação vigente.**

---

<div align="center">

**DFe.NET** - Simplificando a integração fiscal eletrônica no Brasil 🇧🇷

[⭐ Star no GitHub](https://github.com/seu-usuario/DFe.NET) | [📦 NuGet](https://www.nuget.org/packages/DFe.NFe) | [📖 Documentação](https://seu-usuario.github.io/DFe.NET)

</div>
