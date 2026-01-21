# 📋 Padrões de Desenvolvimento - Biblioteca DFe.NET

Este documento define os padrões **obrigatórios e inegociáveis** para desenvolvimento da biblioteca de documentos fiscais eletrônicos brasileiros (NFe, NFCe, CTe, MDFe, NFCom, NFSe).

**Objetivo:** Criar uma biblioteca moderna, robusta, multiplataforma e totalmente em português para emissão e gestão de documentos fiscais eletrônicos brasileiros.

**Público-alvo:** Desenvolvedores .NET que precisam integrar seus sistemas com a SEFAZ.

**Princípios fundamentais:**
- ✅ 100% Cross-platform (Windows, Linux, macOS, containers)
- ✅ 100% Português (código, comentários, documentação)
- ✅ XSD como única fonte de verdade (validação rigorosa)
- ✅ Zero arredondamento em valores monetários
- ✅ Documentação XML completa (públicos E privados)
- ✅ Testes abrangentes por módulo
- ✅ .NET 10 exclusivo (sem retrocompatibilidade)


## 🎯 Escopo do Projeto

### Documentos Fiscais Suportados

| Documento            | Modelo | Layouts Suportados   | Schema Atual           | Status      |
| -------------------- | ------ | -------------------- | ---------------------- | ----------- |
| **NF-e**             | 55     | 4.00, 3.10           | Pacote 010b v1.30      | ✅ Produção |
| **NFC-e**            | 65     | 4.00, 3.10           | Pacote 010b v1.30      | ✅ Produção |
| **CT-e**             | 57     | 4.00, 3.00           | NT 2025.001            | ✅ Produção |
| **MDFe**             | 58     | 3.00                 | Layout 3.00            | ✅ Produção |
| **NFCom**            | 62     | 1.00                 | NT 2025.x              | ✅ Produção |
| **NFS-e (nacional)** | —      | 1.02, 1.01, 1.00     | Padrão Nacional v1.02  | ✅ Produção |

**Observação:** Layouts antigos (NFe 2.00, CTe 2.00, MDFe 2.00) **NÃO serão suportados** - foram descontinuados pela SEFAZ.

### Funcionalidades Core

```
✅ Criação e validação de XMLs (conforme XSD SEFAZ)
✅ Assinatura digital (certificado A1 .pfx)
✅ Envio para autorização (SEFAZ)
✅ Consulta de protocolo
✅ Inutilização de numeração
✅ Eventos (Cancelamento, Carta Correção, Manifestação, etc.)
✅ Validação contra schemas XSD em runtime
✅ Geração de chave de acesso
✅ Cálculo de dígito verificador
✅ Logging estruturado de todas as operações
✅ Auditoria completa (retenção 7 anos - conformidade fiscal)
✅ Retry e circuit breaker (resiliência)
✅ Suporte a múltiplas UFs e ambientes (produção/homologação)
```

### Funcionalidades Fora do Escopo Inicial

```
⏸️ Geração de DANFE/DACTE/DAMDFE (PDF) - implementar posteriormente
⏸️ Integração com bancos de dados - responsabilidade do consumidor
⏸️ Interface gráfica (GUI) - biblioteca apenas
⏸️ API REST - biblioteca apenas (consumidor pode criar sua API)
⏸️ Certificados A3 (SmartCard/Token) - apenas A1 inicialmente
⏸️ CI/CD e DevOps - implementar posteriormente
```

---

## 🌐 Idioma - 100% Português

### Regra Geral: Português Obrigatório

**TUDO em português, sem exceções**, exceto:
- Palavras-chave da linguagem C# (`class`, `public`, `async`, etc.)
- Nomes de bibliotecas externas (`HttpClient`, `XmlSerializer`, etc.)
- APIs do .NET (`Task`, `CancellationToken`, `ILogger`, etc.)

**Exemplos:**

```csharp
// ✅ CORRETO - 100% português
namespace SeuProjeto.DFe.NFe.V4.Models;

/// <summary>
/// C01 - Identificação do emitente da Nota Fiscal Eletrônica.
/// Contém dados cadastrais completos do emitente conforme cadastro na SEFAZ.
/// Conforme Manual de Integração Contribuinte NFe versão 4.00, seção C.
/// </summary>
public class Emitente
{
    /// <summary>
    /// C02 - CNPJ do emitente.
    /// Tipo: TCnpj (xs:string com pattern "\d{14}").
    /// Tamanho: Exatamente 14 dígitos numéricos.
    /// Obrigatoriedade: Condicional - obrigatório quando não informado CPF (C02a).
    /// Validação: Deve ser CNPJ válido conforme algoritmo de dígito verificador.
    /// Conforme manual NFe v4.00 item C02.
    /// </summary>
    [XmlElement("CNPJ")]
    public string? Cnpj { get; set; }
    
    /// <summary>
    /// Valida se CNPJ informado possui formato e dígitos verificadores corretos.
    /// Aplica algoritmo oficial de validação de CNPJ da Receita Federal.
    /// </summary>
    /// <returns>True se CNPJ é válido, False caso contrário</returns>
    private bool ValidarCnpj()
    {
        // Lógica de validação de CNPJ
        return true;
    }
}

// ✅ CORRETO - Exceções em português
public class NFeNaoEncontradaExcecao : Exception
{
    public NFeNaoEncontradaExcecao(string chaveAcesso) 
        : base($"NFe com chave de acesso {chaveAcesso} não foi encontrada")
    {
    }
}

// ✅ CORRETO - Logs em português
_logger.LogInformation(
    "Enviando NFe para autorização. Chave={ChaveAcesso}, UF={UF}, Ambiente={Ambiente}",
    nfe.ChaveAcesso,
    nfe.Uf,
    nfe.Ambiente
);

// ✅ CORRETO - Commits em português
git commit -m "feat: adiciona validação de CNPJ no emitente"
git commit -m "fix: corrige timeout em requisições para SEFAZ"
git commit -m "refactor: reorganiza validadores por domínio"

// ❌ INCORRETO - Inglês
public class Invoice
{
    public string TaxId { get; set; }
    
    private bool ValidateTaxId()
    {
        // Validate tax ID
        return true;
    }
}

_logger.LogInformation("Sending invoice to authorization");
git commit -m "feat: add tax id validation"
```

### Glossário de Termos (Português ↔ Inglês)

Para manter consistência, sempre usar os termos em português:

| Português (✅ USAR)       | Inglês (❌ EVITAR)       |
| ------------------------- | ------------------------ |
| NotaFiscalEletronica      | ElectronicInvoice        |
| Emitente                  | Issuer/Sender            |
| Destinatario              | Recipient/Receiver       |
| ChaveAcesso               | AccessKey                |
| Autorizacao               | Authorization            |
| Cancelamento              | Cancellation             |
| Inutilizacao              | Disablement              |
| Protocolo                 | Protocol                 |
| Repositorio               | Repository               |
| Servico                   | Service                  |
| Validador                 | Validator                |
| Integrador                | Integrator               |
| Processador               | Processor                |
| Excecao                   | Exception                |

---

## 🔢 Valores Monetários - ZERO Arredondamento

### Regra Crítica: Precisão Absoluta

**SEMPRE** usar `decimal` com precisão máxima. **NUNCA** arredondar valores em cálculos intermediários.

**Razão:** Conformidade fiscal exige precisão total. Qualquer arredondamento pode gerar divergência com SEFAZ.

```csharp
// ✅ CORRETO - decimal sem arredondamento
public class ItemNotaFiscal
{
    /// <summary>
    /// I10 - Valor unitário de comercialização do item.
    /// Tipo: TDec_1302 (decimal 13 dígitos, 2 decimais).
    /// Valor exato SEM arredondamento conforme emitido.
    /// Conforme manual NFe v4.00 item I10.
    /// </summary>
    public decimal ValorUnitario { get; set; }
    
    /// <summary>
    /// I11 - Quantidade comercial do item.
    /// Tipo: TDec_1104 (decimal 11 dígitos, 4 decimais).
    /// Quantidade exata conforme negociada.
    /// Conforme manual NFe v4.00 item I11.
    /// </summary>
    public decimal Quantidade { get; set; }
    
    /// <summary>
    /// Calcula valor total do item mantendo precisão completa.
    /// NUNCA arredondar em cálculos intermediários.
    /// SEFAZ valida com precisão total.
    /// </summary>
    public decimal CalcularValorTotal()
    {
        // ✅ Multiplicação direta, sem arredondamento
        return ValorUnitario * Quantidade;
    }
    
    /// <summary>
    /// Calcula valor com desconto mantendo precisão total.
    /// </summary>
    public decimal CalcularValorComDesconto(decimal percentualDesconto)
    {
        // ✅ Cálculos diretos, sem arredondamento
        decimal valorTotal = ValorUnitario * Quantidade;
        decimal valorDesconto = valorTotal * percentualDesconto / 100;
        return valorTotal - valorDesconto;
    }
}

// ✅ CORRETO - Operações com decimal
public class CalculadoraTributos
{
    /// <summary>
    /// Calcula ICMS mantendo precisão total conforme legislação.
    /// Base de cálculo e alíquota devem resultar em valor exato.
    /// </summary>
    public decimal CalcularIcms(
        decimal baseCalculo, 
        decimal aliquota)
    {
        // ✅ Tipo explícito em cálculos fiscais para clareza
        decimal valorIcms = baseCalculo * aliquota / 100;
        
        // ✅ Retorna valor EXATO, sem arredondamento
        return valorIcms;
    }
}

// ✅ CORRETO - Comparação de valores monetários
public bool ValoresIguais(decimal valor1, decimal valor2)
{
    // ✅ Comparação direta, sem tolerância
    // SEFAZ exige valores exatamente iguais
    return valor1 == valor2;
}

// ❌ INCORRETO - NUNCA fazer isso
public decimal CalcularTotal()
{
    // ❌ PROIBIDO: usar double para valores monetários
    double valor = 1000.50;
    
    // ❌ PROIBIDO: arredondar valores
    return Math.Round((decimal)valor * 1.1m, 2);
    
    // ❌ PROIBIDO: usar float
    float valorFloat = 100.50f;
}

// ❌ INCORRETO - Comparação com tolerância
public bool ValoresProximos(decimal valor1, decimal valor2)
{
    // ❌ NUNCA use tolerância em valores fiscais
    decimal diferenca = Math.Abs(valor1 - valor2);
    return diferenca <= 0.01m;
}
```

### Conversão e Serialização

```csharp
// ✅ CORRETO - Serialização XML mantendo precisão
public class ItemNotaFiscalDto
{
    /// <summary>
    /// I10 - Valor unitário (serializado com precisão total)
    /// </summary>
    [XmlElement("vUnCom")]
    public decimal ValorUnitario { get; set; }
    
    // XmlSerializer mantém precisão do decimal automaticamente ✅
}

// ✅ CORRETO - Formatação para exibição (apenas apresentação, não cálculo)
public string FormatarValorParaExibicao(decimal valor)
{
    // Formatar apenas para exibição ao usuário
    // NUNCA use esse valor formatado em cálculos
    return valor.ToString("C2", new CultureInfo("pt-BR"));
    // Exemplo: R$ 1.234,56
}
```

**Resumo da Regra:**
- ✅ `decimal` para TODOS os valores monetários/fiscais
- ❌ NUNCA `double` ou `float`
- ❌ NUNCA `Math.Round()` em cálculos
- ✅ Comparação direta com `==` (sem tolerância)
- ✅ Precisão total em todas as operações

---

## 📅 Datas e Timezone - UTC com Conversão BR

### Regra: Armazenar UTC, Exibir UTC-3

**SEMPRE** armazenar datas em UTC no banco/memória. **SEMPRE** converter para UTC-3 (Horário de Brasília) na apresentação.

**Razão:** Conformidade com legislação fiscal brasileira e documentação SEFAZ.

```csharp
/// <summary>
/// Configuração de timezone padrão do sistema DFe.
/// Todas as operações fiscais seguem horário oficial de Brasília (UTC-3).
/// </summary>
public static class TimeZoneConfiguracao
{
    /// <summary>
    /// Timezone oficial: UTC-3 (Horário de Brasília).
    /// Usado para todas as operações fiscais conforme legislação brasileira.
    /// ID do timezone: "E. South America Standard Time" (Windows/Linux/macOS).
    /// </summary>
    public static readonly TimeZoneInfo TimeZoneBrasil = 
        TimeZoneInfo.FindSystemTimeZoneById("E. South America Standard Time");
    
    /// <summary>
    /// Obtém data/hora atual em horário de Brasília (UTC-3).
    /// Use este método ao invés de DateTime.Now para garantir consistência.
    /// </summary>
    /// <returns>DateTime atual em UTC-3</returns>
    public static DateTime AgoraBrasil() => 
        TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, TimeZoneBrasil);
    
    /// <summary>
    /// Converte data/hora de UTC para horário de Brasília (UTC-3).
    /// </summary>
    /// <param name="utcDate">Data em UTC</param>
    /// <returns>Data convertida para UTC-3</returns>
    public static DateTime ParaBrasil(DateTime utcDate)
    {
        if (utcDate.Kind != DateTimeKind.Utc)
        {
            throw new ArgumentException(
                "Data deve estar em UTC. Use DateTime.UtcNow ou especifique DateTimeKind.Utc",
                nameof(utcDate)
            );
        }
        
        return TimeZoneInfo.ConvertTimeFromUtc(utcDate, TimeZoneBrasil);
    }
    
    /// <summary>
    /// Converte data/hora de horário de Brasília (UTC-3) para UTC.
    /// </summary>
    /// <param name="dataBrasil">Data em UTC-3</param>
    /// <returns>Data convertida para UTC</returns>
    public static DateTime ParaUtc(DateTime dataBrasil) => 
        TimeZoneInfo.ConvertTimeToUtc(dataBrasil, TimeZoneBrasil);
}

// ✅ CORRETO - Uso em entidades
public class NotaFiscalEletronica
{
    /// <summary>
    /// B09 - Data e hora de emissão da NFe.
    /// Armazenada em UTC no banco, exibida em UTC-3 ao usuário.
    /// Formato: AAAA-MM-DDTHH:MM:SS-03:00 (com timezone).
    /// Conforme manual NFe v4.00 item B09.
    /// </summary>
    public DateTime DataEmissao { get; set; }
    
    /// <summary>
    /// Retorna data de emissão em horário de Brasília (UTC-3).
    /// Use esta propriedade para exibição ao usuário.
    /// </summary>
    public DateTime DataEmissaoLocal => 
        TimeZoneConfiguracao.ParaBrasil(DataEmissao);
    
    /// <summary>
    /// Define data de emissão usando horário atual de Brasília.
    /// Armazena internamente em UTC.
    /// </summary>
    public void DefinirDataEmissaoAtual()
    {
        // ✅ Armazena em UTC
        DataEmissao = DateTime.UtcNow;
    }
}

// ✅ CORRETO - Formatação para XML SEFAZ
public class NFeSerializador
{
    /// <summary>
    /// Formata data/hora para formato aceito pela SEFAZ.
    /// Formato: AAAA-MM-DDTHH:MM:SS-03:00
    /// </summary>
    public string FormatarDataParaSefaz(DateTime dataUtc)
    {
        DateTime dataBrasil = TimeZoneConfiguracao.ParaBrasil(dataUtc);
        
        // Formato ISO 8601 com timezone -03:00
        return dataBrasil.ToString("yyyy-MM-ddTHH:mm:ss-03:00");
    }
}

// ❌ INCORRETO - Não usar DateTime.Now
public class NotaFiscalEletronica
{
    public void Criar()
    {
        // ❌ DateTime.Now usa timezone do servidor (pode variar)
        DataCriacao = DateTime.Now;
    }
}

// ❌ INCORRETO - Não ignorar timezone
public bool EhHoje(DateTime data)
{
    // ❌ Comparação errada - não considera timezone
    return data.Date == DateTime.Now.Date;
}
```

### Constantes de Tempo

```csharp
/// <summary>
/// Constantes de configuração temporal para operações fiscais.
/// </summary>
public static class ConstantesTempo
{
    /// <summary>
    /// ID do timezone oficial do sistema (UTC-3).
    /// Compatível com Windows, Linux e macOS.
    /// </summary>
    public const string TIMEZONE_ID = "E. South America Standard Time";
    
    /// <summary>
    /// Formato de data padrão para exibição (DD/MM/AAAA).
    /// </summary>
    public const string FORMATO_DATA = "dd/MM/yyyy";
    
    /// <summary>
    /// Formato de data/hora completo (DD/MM/AAAA HH:MM:SS).
    /// </summary>
    public const string FORMATO_DATA_HORA = "dd/MM/yyyy HH:mm:ss";
    
    /// <summary>
    /// Formato ISO 8601 com timezone para SEFAZ.
    /// </summary>
    public const string FORMATO_SEFAZ = "yyyy-MM-ddTHH:mm:ss-03:00";
}
```

- ✅ Armazenar: `DateTime.UtcNow` (UTC)
- ✅ Exibir: Converter para UTC-3 via `TimeZoneConfiguracao.ParaBrasil()`
- ❌ NUNCA usar `DateTime.Now` (timezone do servidor)
- ✅ Sempre especificar `DateTimeKind.Utc`
- ✅ Formatar para SEFAZ: ISO 8601 com `-03:00`

---

## 📁 Organização de Arquivos - Uma Classe Por Arquivo

### Regra: Responsabilidade Única + Navegação Fácil

**Cada classe, interface, record, enum ou struct deve estar em seu próprio arquivo.**

**Razão:** Facilita navegação, refatoração, code review, testes unitários, e seguir princípio de Responsabilidade Única (SRP).

```csharp
// ✅ CORRETO - Arquivos separados

// Emitente.cs
namespace SeuProjeto.DFe.NFe.V4.Models;

/// <summary>
/// C01 - Identificação do emitente da Nota Fiscal Eletrônica.
/// </summary>
public class Emitente
{
    [XmlElement("CNPJ")]
    public string? Cnpj { get; set; }
    
    [XmlElement("xNome")]
    public string RazaoSocial { get; set; } = string.Empty;
}

// Destinatario.cs
namespace SeuProjeto.DFe.NFe.V4.Models;

/// <summary>
/// E01 - Identificação do destinatário da Nota Fiscal Eletrônica.
/// </summary>
public class Destinatario
{
    [XmlElement("CNPJ")]
    public string? Cnpj { get; set; }
    
    [XmlElement("CPF")]
    public string? Cpf { get; set; }
}

// ❌ INCORRETO - Múltiplas classes em um arquivo
// Entidades.cs
public class Emitente { }
public class Destinatario { }
public class Produto { }
```

### Estrutura de Diretórios - Namespace por Versão

```
src/
├── SeuProjeto.DFe.Core/                    # Componentes compartilhados
│   ├── Abstratos/                          # Interfaces principais
│   │   ├── IServicoSefaz.cs
│   │   ├── IValidadorXsd.cs
│   │   ├── IAssinadorDigital.cs
│   │   ├── IRepositorioChaveAcesso.cs
│   │   └── IProvedorEnderecosSefaz.cs
│   ├── Excecoes/                           # Hierarquia de exceções
│   │   ├── DFeException.cs                 # Base abstrata
│   │   ├── ValidationException.cs
│   │   ├── SchemaValidationException.cs
│   │   ├── SefazCommunicationException.cs
│   │   ├── CertificateException.cs
│   │   └── SignatureException.cs
│   ├── Modelos/                            # Modelos base
│   │   ├── ChaveAcesso.cs                  # Chave 44 dígitos
│   │   ├── Protocolo.cs
│   │   ├── ResultadoAutorizacao.cs
│   │   ├── InformacaoAdicional.cs
│   │   └── Assinatura.cs                   # Estrutura <Signature>
│   ├── Validadores/                        # Validadores reutilizáveis
│   │   ├── ValidadorCnpj.cs
│   │   ├── ValidadorCpf.cs
│   │   ├── ValidadorInscricaoEstadual.cs
│   │   ├── ValidadorChaveAcesso.cs
│   │   └── CalculadoraDigitoVerificador.cs
│   └── Configuracao/                       # Configuração global
│       ├── TimeZoneConfiguracao.cs         # UTC-3 Brasil
│       ├── ConstantesDFe.cs                # Constantes gerais
│       └── VersaoSchema.cs                 # Enumeração de versões
│
├── SeuProjeto.DFe.NFe/                     # NF-e (Modelo 55/65)
│   ├── V4/                                 # Layout 4.00 (ATUAL)
│   │   ├── Modelos/
│   │   │   ├── NotaFiscalEletronica.cs     # Estrutura completa <NFe>
│   │   │   ├── InformacoesNFe.cs           # <infNFe>
│   │   │   ├── Identificacao.cs            # B01 - <ide>
│   │   │   ├── Emitente.cs                 # C01 - <emit>
│   │   │   ├── Destinatario.cs             # E01 - <dest>
│   │   │   ├── ItemNFe.cs                  # H01 - <det>
│   │   │   ├── Produto.cs                  # I01 - <prod>
│   │   │   ├── Imposto.cs                  # M01 - <imposto>
│   │   │   ├── Icms.cs                     # N01 - <ICMS>
│   │   │   ├── Ipi.cs                      # O01 - <IPI>
│   │   │   ├── Pis.cs                      # Q01 - <PIS>
│   │   │   ├── Cofins.cs                   # S01 - <COFINS>
│   │   │   ├── Total.cs                    # W01 - <total>
│   │   │   ├── Transporte.cs               # X01 - <transp>
│   │   │   ├── Cobranca.cs                 # Y01 - <cobr>
│   │   │   ├── Pagamento.cs                # YA01 - <pag>
│   │   │   ├── InformacoesAdicionais.cs    # Z01 - <infAdic>
│   │   │   └── ...
│   │   ├── Validadores/                    # FluentValidation
│   │   │   ├── ValidadorNotaFiscalEletronica.cs
│   │   │   ├── ValidadorEmitente.cs
│   │   │   ├── ValidadorDestinatario.cs
│   │   │   ├── ValidadorProduto.cs
│   │   │   └── ...
│   │   ├── Servicos/                       # Serviços SOAP
│   │   │   ├── ServicoAutorizacaoNFe.cs    # NFeAutorizacao4
│   │   │   ├── ServicoConsultaNFe.cs       # NFeConsultaProtocolo4
│   │   │   ├── ServicoConsultaStatusSefaz.cs # NFeStatusServico4
│   │   │   ├── ServicoCancelamentoNFe.cs   # RecepcaoEvento4 (tipo 110111)
│   │   │   ├── ServicoCartaCorrecaoNFe.cs  # RecepcaoEvento4 (tipo 110110)
│   │   │   ├── ServicoInutilizacaoNFe.cs   # NFeInutilizacao4
│   │   │   └── ...
│   │   ├── Enumeracoes/
│   │   │   ├── ModeloDocumento.cs          # 55=NFe, 65=NFCe
│   │   │   ├── TipoEmissao.cs              # 1=Normal, 9=Contingência
│   │   │   ├── FinalidadeEmissao.cs        # 1=Normal, 3=Ajuste, 4=Devolução
│   │   │   ├── TipoOperacao.cs             # 0=Entrada, 1=Saída
│   │   │   └── ...
│   │   └── Schemas/                        # XSDs embedded resources
│   │       ├── nfe_v4.00.xsd
│   │       ├── procNFe_v4.00.xsd
│   │       ├── envNFe_v4.00.xsd
│   │       ├── retEnvNFe_v4.00.xsd
│   │       ├── consSitNFe_v4.00.xsd
│   │       ├── retConsSitNFe_v4.00.xsd
│   │       ├── inutNFe_v4.00.xsd
│   │       └── ...
│   │
│   └── V3/                                 # Layout 3.10 (LEGADO)
│       ├── Modelos/
│       ├── Validadores/
│       ├── Servicos/
│       └── Schemas/
│
├── SeuProjeto.DFe.CTe/                     # CT-e (Modelo 57)
│   ├── V4/                                 # Layout 4.00 (ATUAL)
│   │   ├── Modelos/
│   │   │   ├── ConhecimentoTransporte.cs
│   │   │   ├── InformacoesCTe.cs
│   │   │   ├── Remetente.cs
│   │   │   ├── Destinatario.cs
│   │   │   ├── Tomador.cs
│   │   │   ├── Expedidor.cs
│   │   │   ├── Recebedor.cs
│   │   │   ├── Carga.cs
│   │   │   ├── Componentes.cs              # Frete, seguro, pedágio
│   │   │   └── ...
│   │   ├── Validadores/
│   │   ├── Servicos/
│   │   └── Schemas/
│   │
│   └── V3/                                 # Layout 3.00 (LEGADO)
│       └── ...
│
├── SeuProjeto.DFe.MDFe/                    # MDF-e (Modelo 58)
│   └── V3/                                 # Layout 3.00 (ATUAL)
│       ├── Modelos/
│       │   ├── ManifestoDocumentoFiscal.cs
│       │   ├── InformacoesMDFe.cs
│       │   ├── Emitente.cs
│       │   ├── Motorista.cs
│       │   ├── Veiculo.cs
│       │   ├── Seguro.cs
│       │   ├── Lacre.cs
│       │   └── ...
│       ├── Validadores/
│       ├── Servicos/
│       └── Schemas/
│
├── SeuProjeto.DFe.NFCom/                   # NFCom (Modelo 62)
│   └── V1/                                 # Layout 1.00 (ATUAL)
│       ├── Modelos/
│       │   ├── NotaFiscalComunicacao.cs
│       │   ├── Prestador.cs
│       │   ├── Tomador.cs
│       │   ├── Assinante.cs
│       │   ├── ServicosComunicacao.cs
│       │   └── ...
│       ├── Validadores/
│       ├── Servicos/
│       └── Schemas/
│
├── SeuProjeto.DFe.NFSe/                    # NFS-e Padrão Nacional
│   ├── V1_02/                              # Layout 1.02 (ATUAL)
│   │   ├── Modelos/
│   │   │   ├── NotaFiscalServico.cs
│   │   │   ├── Prestador.cs
│   │   │   ├── Tomador.cs
│   │   │   ├── Servico.cs
│   │   │   ├── ValoresServico.cs
│   │   │   └── ...
│   │   ├── Validadores/
│   │   ├── Servicos/
│   │   └── Schemas/
│   ├── V1_01/                              # Layout 1.01
│   └── V1_00/                              # Layout 1.00
│
└── SeuProjeto.DFe.Infraestrutura/          # Infraestrutura cross-cutting
    ├── Http/                               # Comunicação HTTP
    │   ├── ClienteHttpSefaz.cs             # Cliente base para SEFAZ
    │   ├── FabricaClienteHttpSefaz.cs
    │   ├── ManipuladorMensagemSoap.cs      # Envelope SOAP 1.2
    │   └── PoliticasPolly.cs               # Retry, timeout, circuit breaker
    ├── Xml/                                # Serialização XML
    │   ├── SerializadorXml.cs
    │   ├── DeserializadorXml.cs
    │   ├── GerenciadorNamespace.cs         # Gerencia namespaces SEFAZ
    │   └── ExtensoesXml.cs
    ├── Seguranca/                          # Segurança e assinatura
    │   ├── AssinadorDigital.cs             # Assina XMLs (SHA-1/SHA-256)
    │   ├── CertificadoDigital.cs           # Gerencia certificados .pfx
    │   ├── ValidadorAssinatura.cs
    │   └── CalculadoraHash.cs
    ├── Validacao/                          # Validação XSD
    │   ├── ValidadorXsd.cs
    │   ├── ProvedorSchemaEmbutido.cs       # Carrega XSDs embedded
    │   └── ResultadoValidacao.cs
    ├── Auditoria/                          # Auditoria
    │   ├── ServicoAuditoria.cs             # Registra TODAS as operações
    │   ├── EnriquecedorLog.cs
    │   └── MascaradorDadosSensiveis.cs     # Mascara CPF/CNPJ em logs
    └── Resiliencia/                        # Resiliência
        ├── PoliticaRetry.cs                # Polly retry
        ├── PoliticaTimeout.cs              # Polly timeout
        └── PoliticaCircuitBreaker.cs       # Polly circuit breaker
```

**Nomenclatura de Arquivos:**
- Nome do arquivo = Nome da classe/interface/enum (PascalCase)
- Interfaces: `I{Nome}.cs` (ex: `IServicoSefaz.cs`)
- Classes abstratas: `{Nome}Base.cs` (ex: `ValidadorBase.cs`)
- Exceções: `{Nome}Exception.cs` (ex: `ValidationException.cs`)

---

## 🏷️ Convenções de Nomenclatura - Padrão Híbrido

### Característica Única do Projeto DFe

Este projeto possui necessidade **híbrida**:
- **Código C#**: Convenções idiomáticas (.NET) em **português**
- **XML SEFAZ**: Tags oficiais (não modificar)

#### Classes e Propriedades: PascalCase em Português

```csharp
// ✅ CORRETO - Nomes descritivos em PascalCase português
namespace SeuProjeto.DFe.NFe.V4.Modelos;

/// <summary>
/// C01 - Identificação do emitente da Nota Fiscal Eletrônica.
/// Representa tag &lt;emit&gt; conforme schema SEFAZ nfe_v4.00.xsd.
/// Conforme Manual de Integração Contribuinte NFe versão 4.00, seção C.
/// </summary>
public class Emitente
{
    /// <summary>
    /// C02 - CNPJ do emitente.
    /// Tag XML: &lt;CNPJ&gt;
    /// Tipo: TCnpj (14 dígitos numéricos).
    /// Obrigatoriedade: Condicional - obrigatório quando não informado CPF (C02a).
    /// Validação: Aplicar algoritmo de dígito verificador CNPJ.
    /// </summary>
    [XmlElement("CNPJ")]
    public string? Cnpj { get; set; }
    
    /// <summary>
    /// C03 - Razão social ou nome empresarial do emitente.
    /// Tag XML: &lt;xNome&gt;
    /// Tipo: TString (mínimo 2, máximo 60 caracteres).
    /// Obrigatoriedade: Obrigatório.
    /// </summary>
    [XmlElement("xNome")]
    public string RazaoSocial { get; set; } = string.Empty;
    
    /// <summary>
    /// C04 - Nome fantasia do emitente.
    /// Tag XML: &lt;xFant&gt;
    /// Tipo: TString (mínimo 1, máximo 60 caracteres).
    /// Obrigatoriedade: Opcional.
    /// </summary>
    [XmlElement("xFant")]
    public string? NomeFantasia { get; set; }
    
    /// <summary>
    /// C06 - Inscrição Estadual do emitente.
    /// Tag XML: &lt;IE&gt;
    /// Tipo: TIe (2 a 14 caracteres).
    /// Obrigatoriedade: Obrigatório (informar ISENTO para contribuintes isentos).
    /// </summary>
    [XmlElement("IE")]
    public string InscricaoEstadual { get; set; } = string.Empty;
    
    /// <summary>
    /// Endereço completo do emitente.
    /// Representa tag &lt;enderEmit&gt;.
    /// </summary>
    [XmlElement("enderEmit")]
    public EnderecoEmitente Endereco { get; set; } = new();
}
```

**Justificativa Híbrida:**
- **Propriedade C#**: `RazaoSocial` (descritivo, idiomático)
- **Atributo XML**: `[XmlElement("xNome")]` (exato conforme XSD SEFAZ)
- **Comentário**: Mapeia ID oficial SEFAZ (`C03`) e tag (`<xNome>`)

#### Atributos XML: NUNCA Modificar

```csharp
// ✅ CORRETO - Tags EXATAMENTE como XSD SEFAZ
[XmlElement("CNPJ")]              // Tag oficial
public string? Cnpj { get; set; }

[XmlElement("xNome")]             // Tag oficial (x = texto)
public string RazaoSocial { get; set; }

[XmlElement("vProd")]             // Tag oficial (v = valor)
public decimal ValorProduto { get; set; }

[XmlElement("qCom")]              // Tag oficial (q = quantidade)
public decimal Quantidade { get; set; }

// ❌ INCORRETO - NUNCA alterar tags SEFAZ
[XmlElement("CompanyTaxId")]      // ❌ SEFAZ rejeitará
[XmlElement("CNPJ_Empresa")]      // ❌ SEFAZ rejeitará
[XmlElement("cnpj")]              // ❌ Case-sensitive, SEFAZ rejeitará
```

**Razão:** SEFAZ valida XML contra XSD oficial. Qualquer diferença causa **Rejeição 215** (Falha no schema XML).

#### Namespaces: PascalCase + Versionamento Explícito

```csharp
// ✅ CORRETO - Hierárquico com versão explícita
namespace SeuProjeto.DFe.NFe.V4.Modelos;
namespace SeuProjeto.DFe.NFe.V4.Validadores;
namespace SeuProjeto.DFe.NFe.V4.Servicos;
namespace SeuProjeto.DFe.NFe.V3.Modelos;       // Layout 3.10 (coexiste)
namespace SeuProjeto.DFe.CTe.V4.Modelos;
namespace SeuProjeto.DFe.CTe.V3.Modelos;
namespace SeuProjeto.DFe.MDFe.V3.Modelos;
namespace SeuProjeto.DFe.Core.Abstratos;

// ❌ INCORRETO - Sem versionamento
namespace SeuProjeto.DFe.NFe.Modelos;           // ❌ Qual versão? Conflito!
namespace SeuProjeto.NFe4;                       // ❌ Não idiomático
namespace SeuProjeto.DFe.NFe.V4_00.Models;      // ❌ Underscore em namespace
```

**Razão:** Múltiplas versões devem coexistir (empresas podem usar layouts diferentes).

#### Métodos: PascalCase + Verbos em Português

```csharp
// ✅ CORRETO - Verbo + Substantivo em português
public async Task<Protocolo> AutorizarNFeAsync(
    NotaFiscalEletronica nfe, 
    CancellationToken ct)

public async Task<Protocolo> ConsultarProtocoloAsync(
    string chaveAcesso, 
    CancellationToken ct)

public async Task<bool> CancelarNFeAsync(
    string chaveAcesso, 
    string justificativa, 
    CancellationToken ct)

public string GerarChaveAcesso()
public int CalcularDigitoVerificador()
public decimal CalcularValorTotalNFe()
public bool ValidarCnpj(string cnpj)
public XmlDocument AssinarXml(XmlDocument xml)

// ❌ INCORRETO
public async Task<Protocolo> AuthorizeAsync()       // ❌ Inglês
public async Task<Protocolo> Process()              // ❌ Verbo genérico
public decimal Total()                              // ❌ Falta verbo
public async Task Enviar()                          // ❌ Falta "Async"
```

#### Variáveis Locais: camelCase

```csharp
// ✅ CORRETO - camelCase
public async Task ProcessarNFeAsync(
    NotaFiscalEletronica nfe, 
    CancellationToken ct)
{
    string chaveAcesso = nfe.GerarChaveAcesso();
    decimal valorTotal = nfe.CalcularValorTotal();
    bool assinada = nfe.PossuiAssinaturaDigital();
    int quantidadeItens = nfe.Itens.Count;
    TimeSpan tempoProcessamento = DateTime.UtcNow - nfe.DataCriacao;
    
    XmlDocument xmlAssinado = await AssinarXmlAsync(nfe.ToXml(), ct);
}

// ❌ INCORRETO
string ChaveAcesso;        // ❌ PascalCase em variável local
string _chaveAcesso;       // ❌ Underscore é apenas para campos privados
```

#### Campos Privados: _camelCase

```csharp
// ✅ CORRETO - Underscore + camelCase
public class ServicoAutorizacaoNFe : IServicoAutorizacaoNFe
{
    private readonly IHttpClientFactory _httpClientFactory;
    private readonly ILogger<ServicoAutorizacaoNFe> _logger;
    private readonly IAssinadorDigital _assinadorDigital;
    private readonly IValidadorXsd _validadorXsd;
    private readonly IProviderEnderecosSefaz _providerEnderecos;
    
    public ServicoAutorizacaoNFe(
        IHttpClientFactory httpClientFactory,
        ILogger<ServicoAutorizacaoNFe> logger,
        IAssinadorDigital assinadorDigital,
        IValidadorXsd validadorXsd,
        IProviderEnderecosSefaz providerEnderecos)
    {
        _httpClientFactory = httpClientFactory;
        _logger = logger;
        _assinadorDigital = assinadorDigital;
        _validadorXsd = validadorXsd;
        _providerEnderecos = providerEnderecos;
    }
}

// ❌ INCORRETO
private ILogger logger;           // ❌ Sem underscore
private ILogger m_logger;         // ❌ Notação húngara
private ILogger _Logger;          // ❌ PascalCase após underscore
```

#### Constantes: UPPER_SNAKE_CASE

```csharp
// ✅ CORRETO - UPPER_SNAKE_CASE
public static class ConstantesNFe
{
    /// <summary>
    /// Timeout padrão para requisições SOAP à SEFAZ (30 segundos).
    /// </summary>
    public const int TIMEOUT_SEFAZ_SEGUNDOS = 30;
    
    /// <summary>
    /// Máximo de tentativas em caso de falha de comunicação (3 tentativas).
    /// </summary>
    public const int MAX_TENTATIVAS_AUTORIZACAO = 3;
    
    /// <summary>
    /// Delay entre tentativas de retry (2 segundos).
    /// </summary>
    public const int DELAY_ENTRE_TENTATIVAS_SEGUNDOS = 2;
    
    /// <summary>
    /// Namespace XML da NFe versão 4.00.
    /// </summary>
    public const string NAMESPACE_NFE_V4 = "http://www.portalfiscal.inf.br/nfe";
    
    /// <summary>
    /// Tamanho exato da chave de acesso (44 dígitos).
    /// </summary>
    public const int TAMANHO_CHAVE_ACESSO = 44;
    
    /// <summary>
    /// Tamanho do CNPJ (14 dígitos).
    /// </summary>
    public const int TAMANHO_CNPJ = 14;
    
    /// <summary>
    /// Tamanho do CPF (11 dígitos).
    /// </summary>
    public const int TAMANHO_CPF = 11;
}

// ❌ INCORRETO
public const int TimeoutSefaz = 30;                     // ❌ PascalCase
public const string NamespaceNfeV4 = "...";             // ❌ PascalCase
public const int TAMANHO-CHAVE-ACESSO = 44;             // ❌ Hífen não permitido
```

#### Parâmetros: camelCase

```csharp
// ✅ CORRETO - camelCase
public async Task<Protocolo> AutorizarNFeAsync(
    NotaFiscalEletronica nfe,
    AmbienteSefaz ambiente,
    CertificadoDigital certificado,
    CancellationToken cancellationToken)
{
    // Implementação
}

// ❌ INCORRETO
public async Task<Protocolo> AutorizarNFeAsync(
    NotaFiscalEletronica Nfe,                         // ❌ PascalCase
    AmbienteSefaz _ambiente,                          // ❌ Underscore
    CertificadoDigital pCertificado)                 // ❌ Notação húngara
{
}
```

#### Interfaces: I{Nome}

```csharp
// ✅ CORRETO - Prefixo I + PascalCase
public interface IServicoSefaz
{
    Task<Protocolo> AutorizarAsync(string xml, CancellationToken ct);
}

public interface IValidadorXsd
{
    ValidationResult Validar(string xml, string caminhoXsd);
}

public interface IAssinadorDigital
{
    XmlDocument Assinar(XmlDocument xml, CertificadoDigital certificado);
}

// ❌ INCORRETO
public interface ServicoSefaz { }              // ❌ Sem prefixo I
public interface ISefazService { }             // ❌ Inglês
```

#### Enums: PascalCase (Tipo e Valores)

```csharp
// ✅ CORRETO - Tipo e valores em PascalCase
namespace SeuProjeto.DFe.NFe.V4.Enums;

/// <summary>
/// B11 - Código do regime tributário do emitente.
/// 1=Simples Nacional, 2=Simples Nacional - excesso, 3=Regime Normal.
/// </summary>
public enum RegimeTributario
{
    /// <summary>
    /// 1 - Simples Nacional
    /// </summary>
    [XmlEnum("1")]
    SimplesNacional = 1,
    
    /// <summary>
    /// 2 - Simples Nacional - excesso de sublimite de receita bruta
    /// </summary>
    [XmlEnum("2")]
    SimplesNacionalExcesso = 2,
    
    /// <summary>
    /// 3 - Regime Normal (Lucro Real, Presumido ou Arbitrado)
    /// </summary>
    [XmlEnum("3")]
    RegimeNormal = 3
}

/// <summary>
/// B12 - Tipo de operação.
/// 0=Entrada, 1=Saída.
/// </summary>
public enum TipoOperacao
{
    /// <summary>
    /// 0 - Entrada
    /// </summary>
    [XmlEnum("0")]
    Entrada = 0,
    
    /// <summary>
    /// 1 - Saída
    /// </summary>
    [XmlEnum("1")]
    Saida = 1
}

// ❌ INCORRETO
public enum TipoOperacao
{
    entrada = 0,           // ❌ camelCase
    SAIDA = 1             // ❌ UPPER_CASE
}
```

### Resumo das Convenções

| Elemento                | Convenção              | Exemplo                                   |
| ----------------------- | ---------------------- | ----------------------------------------- |
| **Namespace**           | PascalCase + Versão    | `SeuProjeto.DFe.NFe.V4.Modelos`           |
| **Classes**             | PascalCase (português) | `NotaFiscalEletronica`                    |
| **Interfaces**          | I + PascalCase         | `IServicoSefaz`                           |
| **Métodos**             | PascalCase (português) | `AutorizarNFeAsync`                       |
| **Propriedades**        | PascalCase (português) | `RazaoSocial`, `ChaveAcesso`              |
| **Atributos XML**       | **SEFAZ (original)**   | `[XmlElement("xNome")]`                   |
| **Campos privados**     | _camelCase             | `_httpClientFactory`, `_logger`           |
| **Variáveis locais**    | camelCase              | `chaveAcesso`, `valorTotal`               |
| **Parâmetros**          | camelCase              | `nfe`, `certificado`, `cancellationToken` |
| **Constantes**          | UPPER_SNAKE_CASE       | `TIMEOUT_SEFAZ_SEGUNDOS`                  |
| **Enums (tipo)**        | PascalCase             | `RegimeTributario`                        |
| **Enums (valores)**     | PascalCase             | `SimplesNacional`                         |

---

## 🔍 var vs Tipo Explícito - Clareza em Contexto Fiscal

### Regra: Tipos Explícitos em Cálculos Fiscais/Monetários

Para **valores monetários, cálculos fiscais e tipos numéricos**, **SEMPRE use tipos explícitos**.

Para operações onde tipo é óbvio (new, cast, chamada), `var` é aceitável.

**Razão:** Conformidade fiscal exige clareza absoluta. Ambiguidade (double? decimal? float?) pode causar erros críticos.

```csharp
// ✅ PREFERIDO - Tipos explícitos em cálculos fiscais
public class CalculadoraImpostos
{
    /// <summary>
    /// Calcula ICMS conforme legislação estadual.
    /// Base de cálculo e alíquota devem resultar em valor exato (sem arredondamento).
    /// </summary>
    public decimal CalcularIcms(
        decimal baseCalculo,
        decimal aliquotaIcms)
    {
        // ✅ Tipo explícito: clareza absoluta (é decimal, não double/float)
        decimal valorIcms = baseCalculo * aliquotaIcms / 100;
        
        _logger.LogInformation(
            "ICMS calculado: BaseCalculo={BaseCalculo}, Aliquota={Aliquota}%, Valor={Valor}",
            baseCalculo,
            aliquotaIcms,
            valorIcms
        );
        
        return valorIcms;
    }
    
    /// <summary>
    /// Calcula valor total da NFe (produtos + frete + seguro + outras - desconto).
    /// </summary>
    public decimal CalcularValorTotalNFe(
        decimal valorProdutos,
        decimal valorFrete,
        decimal valorSeguro,
        decimal outrasDespesas,
        decimal valorDesconto)
    {
        // ✅ Tipos explícitos: transparência em cálculos críticos
        decimal valorTotal = valorProdutos + valorFrete + valorSeguro + outrasDespesas - valorDesconto;
        
        return valorTotal;
    }
}

// ✅ ACEITÁVEL - var quando tipo é óbvio
public async Task<NotaFiscalEletronica> BuscarNFeAsync(
    string chaveAcesso,
    CancellationToken ct)
{
    // ✅ Tipo óbvio pelo nome do método (retorna NFe)
    var nfe = await _repositorio.BuscarPorChaveAsync(chaveAcesso, ct);
    
    // ✅ Tipo óbvio pela instanciação (new)
    var validador = new EmitenteValidator();
    
    // ✅ Tipo óbvio pelo cast
    var resultado = (ResultadoAutorizacao)response.Resultado;
    
    // ✅ Tipo óbvio em LINQ
    var itensValidos = nfe.Itens.Where(i => i.ValorUnitario > 0).ToList();
    
    return nfe;
}

// ❌ EVITAR - var em cálculos fiscais (ambiguidade perigosa)
public decimal CalcularTotal()
{
    // ❌ Tipo não óbvio: é double? decimal? float?
    var valor = 1000.50;          // Compilador infere double
    var percentual = 10.5;        // Compilador infere double
    var total = valor * percentual / 100;   // Resultado: double (ERRADO!)
    
    return (decimal)total;        // Cast necessário indica problema de design
}

// ✅ CORRETO - Mesmo cálculo com tipos explícitos
public decimal CalcularTotal()
{
    decimal valor = 1000.50m;             // Sufixo 'm' = decimal
    decimal percentual = 10.5m;
    decimal total = valor * percentual / 100;
    
    return total;
}
```

**Regra Prática:**
| Situação                                      | Usar var? | Justificativa                              |
| --------------------------------------------- | --------- | ------------------------------------------ |
| Cálculos fiscais/monetários                   | ❌ NÃO     | Tipo explícito evita ambiguidade crítica   |
| Valores numéricos (int, decimal, double)      | ❌ NÃO     | Clareza sobre tipo exato                   |
| Instanciação (new)                            | ✅ SIM     | Tipo óbvio pelo construtor                 |
| Cast explícito                                | ✅ SIM     | Tipo óbvio pelo cast                       |
| Chamada de método retornando tipo claro      | ✅ SIM     | Nome do método deixa tipo óbvio            |
| LINQ (Where, Select, GroupBy, OrderBy)        | ✅ SIM     | Tipo inferido claramente                   |
| Loops foreach                                 | ✅ SIM     | Tipo da coleção é conhecido                |

---

## 🛡️ Guard Clauses - Early Return Pattern

### Regra: Validar Parâmetros no Início

**SEMPRE** usar guard clauses no início dos métodos. Evita aninhamento profundo (arrow code).

**Razão:** Código linear é mais fácil de ler, manter e testar.

```csharp
// ✅ CORRETO - Guard clauses (linear, fácil de ler)
public async Task<Protocolo> AutorizarNFeAsync(
    NotaFiscalEletronica nfe,
    CertificadoDigital certificado,
    CancellationToken ct)
{
    // Guard clauses: validação no início
    ArgumentNullException.ThrowIfNull(nfe);
    ArgumentNullException.ThrowIfNull(certificado);
    ct.ThrowIfCancellationRequested();
    
    // Guard: validação de estado
    if (nfe.Status != StatusNFe.EmEdicao)
    {
        throw new ValidationException(
            $"NFe com chave {nfe.ChaveAcesso} não está em edição. Status atual: {nfe.Status}"
        );
    }
    
    // Guard: validação de certificado
    if (certificado.EstaExpirado())
    {
        throw new CertificateException(
            $"Certificado {certificado.NumeroSerie} expirou em {certificado.DataExpiracao:dd/MM/yyyy}"
        );
    }
    
    // Lógica principal (sem aninhamento)
    _logger.LogInformation(
        "Iniciando autorização de NFe. Chave={ChaveAcesso}",
        nfe.ChaveAcesso
    );
    
    XmlDocument xmlAssinado = await AssinarXmlAsync(nfe, certificado, ct);
    Protocolo protocolo = await EnviarParaSefazAsync(xmlAssinado, ct);
    
    return protocolo;
}

// ❌ INCORRETO - Arrow code (aninhamento profundo, difícil ler)
public async Task<Protocolo> AutorizarNFeAsync(
    NotaFiscalEletronica nfe,
    CertificadoDigital certificado,
    CancellationToken ct)
{
    if (nfe != null)
    {
        if (certificado != null)
        {
            if (nfe.Status == StatusNFe.EmEdicao)
            {
                if (!certificado.EstaExpirado())
                {
                    // Lógica principal enterrada em 4 níveis de aninhamento
                    XmlDocument xmlAssinado = await AssinarXmlAsync(nfe, certificado, ct);
                    Protocolo protocolo = await EnviarParaSefazAsync(xmlAssinado, ct);
                    return protocolo;
                }
                else
                {
                    throw new CertificateException("Certificado expirado");
                }
            }
            else
            {
                throw new ValidationException("NFe não está em edição");
            }
        }
        else
        {
            throw new ArgumentNullException(nameof(certificado));
        }
    }
    else
    {
        throw new ArgumentNullException(nameof(nfe));
    }
}
```

### Guard Clauses com Nullable (.NET 10)

```csharp
// ✅ CORRETO - Guard clauses com helpers .NET
public class ValidadorChaveAcesso
{
    /// <summary>
    /// Valida formato e dígito verificador da chave de acesso.
    /// </summary>
    public bool Validar(string? chaveAcesso)
    {
        // Guard: null ou vazio
        if (string.IsNullOrWhiteSpace(chaveAcesso))
        {
            return false;
        }
        
        // Guard: tamanho inválido
        if (chaveAcesso.Length != ConstantesNFe.TAMANHO_CHAVE_ACESSO)
        {
            return false;
        }
        
        // Guard: não é numérico
        if (!chaveAcesso.All(char.IsDigit))
        {
            return false;
        }
        
        // Lógica principal: valida dígito verificador
        int digitoVerificador = CalcularDigitoVerificador(chaveAcesso[..43]);
        return digitoVerificador == int.Parse(chaveAcesso[43].ToString());
    }
}
```

---

## 🔁 Switch Expressions - Redução de if/else

### Regra: Substituir if/else por switch expressions

Quando há múltiplas condições baseadas em um valor, usar **switch expressions** (C# 8+).

```csharp
// ✅ CORRETO - Switch expression (conciso, claro)
public string ObterDescricaoStatus(StatusNFe status) => status switch
{
    StatusNFe.EmEdicao => "NFe em edição",
    StatusNFe.Assinada => "NFe assinada digitalmente",
    StatusNFe.Autorizada => "NFe autorizada pela SEFAZ",
    StatusNFe.Cancelada => "NFe cancelada",
    StatusNFe.Inutilizada => "Numeração inutilizada",
    StatusNFe.Denegada => "NFe denegada pela SEFAZ",
    _ => throw new ArgumentOutOfRangeException(nameof(status), status, "Status desconhecido")
};

// ❌ INCORRETO - Cadeia de if/else (verboso)
public string ObterDescricaoStatus(StatusNFe status)
{
    if (status == StatusNFe.EmEdicao)
    {
        return "NFe em edição";
    }
    else if (status == StatusNFe.Assinada)
    {
        return "NFe assinada digitalmente";
    }
    else if (status == StatusNFe.Autorizada)
    {
        return "NFe autorizada pela SEFAZ";
    }
    // ... 10 linhas a mais
}

// ✅ CORRETO - Switch expression com pattern matching
public decimal ObterAliquotaIcms(string uf, string ncm) => (uf, ncm) switch
{
    ("SP", "84714100") => 18.00m,
    ("SC", "84714100") => 17.00m,
    ("RS", "84714100") => 18.00m,
    ("SP", _) => 18.00m,              // Alíquota padrão SP
    ("SC", _) => 17.00m,              // Alíquota padrão SC
    ("RS", _) => 18.00m,              // Alíquota padrão RS
    _ => 12.00m                       // Alíquota padrão nacional
};

// ✅ CORRETO - Switch expression para URLs SEFAZ
public string ObterUrlSefaz(string uf, AmbienteSefaz ambiente) => (uf, ambiente) switch
{
    ("SP", AmbienteSefaz.Producao) => "https://nfe.fazenda.sp.gov.br/ws/nfeautorizacao4.asmx",
    ("SP", AmbienteSefaz.Homologacao) => "https://homologacao.nfe.fazenda.sp.gov.br/ws/nfeautorizacao4.asmx",
    ("SC", AmbienteSefaz.Producao) => "https://nfe.svrs.rs.gov.br/ws/NfeAutorizacao/NFeAutorizacao4.asmx",
    ("SC", AmbienteSefaz.Homologacao) => "https://nfe-homologacao.svrs.rs.gov.br/ws/NfeAutorizacao/NFeAutorizacao4.asmx",
    // ... outras UFs
    _ => throw new NotSupportedException($"UF {uf} não suportada no ambiente {ambiente}")
};
```

---

## 🆕 Features .NET 10 - Obrigatórias

### Primary Constructors (C# 12) - OBRIGATÓRIO

**SEMPRE** usar primary constructors em serviços com injeção de dependência.

```csharp
// ✅ CORRETO - Primary constructor (conciso, idiomático .NET 10)
namespace SeuProjeto.DFe.NFe.V4.Servicos;

/// <summary>
/// Serviço de autorização de Notas Fiscais Eletrônicas (NFe).
/// Realiza assinatura digital, validação XSD e envio para SEFAZ.
/// </summary>
public class ServicoAutorizacaoNFe(
    IHttpClientFactory httpClientFactory,
    ILogger<ServicoAutorizacaoNFe> logger,
    IAssinadorDigital assinadorDigital,
    IValidadorXsd validadorXsd,
    IProviderEnderecosSefaz providerEnderecos) : IServicoAutorizacaoNFe
{
    /// <summary>
    /// Autoriza NFe junto à SEFAZ.
    /// </summary>
    public async Task<Protocolo> AutorizarAsync(
        NotaFiscalEletronica nfe,
        CertificadoDigital certificado,
        CancellationToken ct)
    {
        ArgumentNullException.ThrowIfNull(nfe);
        ArgumentNullException.ThrowIfNull(certificado);
        
        logger.LogInformation(
            "Autorizando NFe. Chave={ChaveAcesso}, UF={UF}",
            nfe.ChaveAcesso,
            nfe.Emitente.Endereco.UF
        );
        
        // Assinar XML
        XmlDocument xmlAssinado = assinadorDigital.Assinar(nfe.ToXml(), certificado);
        
        // Validar contra XSD
        ValidationResult validacao = validadorXsd.Validar(xmlAssinado.OuterXml, "nfe_v4.00.xsd");
        if (!validacao.IsValid)
        {
            throw new SchemaValidationException(
                $"XML inválido: {string.Join(", ", validacao.Errors)}"
            );
        }
        
        // Enviar para SEFAZ
        string urlSefaz = providerEnderecos.ObterUrlAutorizacao(nfe.Emitente.Endereco.UF, nfe.Ambiente);
        HttpClient httpClient = httpClientFactory.CreateClient("SEFAZ");
        
        // ... lógica de envio
        
        return new Protocolo();
    }
}

// ❌ INCORRETO - Construtor tradicional (verboso)
public class ServicoAutorizacaoNFe : IServicoAutorizacaoNFe
{
    private readonly IHttpClientFactory _httpClientFactory;
    private readonly ILogger<ServicoAutorizacaoNFe> _logger;
    private readonly IAssinadorDigital _assinadorDigital;
    private readonly IValidadorXsd _validadorXsd;
    private readonly IProviderEnderecosSefaz _providerEnderecos;
    
    public ServicoAutorizacaoNFe(
        IHttpClientFactory httpClientFactory,
        ILogger<ServicoAutorizacaoNFe> logger,
        IAssinadorDigital assinadorDigital,
        IValidadorXsd validadorXsd,
        IProviderEnderecosSefaz providerEnderecos)
    {
        _httpClientFactory = httpClientFactory;
        _logger = logger;
        _assinadorDigital = assinadorDigital;
        _validadorXsd = validadorXsd;
        _providerEnderecos = providerEnderecos;
    }
    
    // ... métodos
}
```

### File-Scoped Namespaces (C# 10) - OBRIGATÓRIO

**SEMPRE** usar file-scoped namespaces (reduz indentação).

```csharp
// ✅ CORRETO - File-scoped namespace
namespace SeuProjeto.DFe.NFe.V4.Modelos;

/// <summary>
/// C01 - Identificação do emitente da NFe.
/// </summary>
public class Emitente
{
    [XmlElement("CNPJ")]
    public string? Cnpj { get; set; }
}

// ❌ INCORRETO - Namespace com chaves (indentação extra)
namespace SeuProjeto.DFe.NFe.V4.Models
{
    public class Emitente
    {
        public string? Cnpj { get; set; }
    }
}
```

### Required Members (C# 11) - OBRIGATÓRIO em DTOs

**SEMPRE** usar `required` para propriedades obrigatórias em DTOs/Models.

```csharp
// ✅ CORRETO - Required members
namespace SeuProjeto.DFe.NFe.V4.Modelos;

/// <summary>
/// C01 - Identificação do emitente da NFe.
/// </summary>
public class Emitente
{
    /// <summary>
    /// C02 - CNPJ do emitente (obrigatório quando não informado CPF).
    /// </summary>
    [XmlElement("CNPJ")]
    public string? Cnpj { get; set; }
    
    /// <summary>
    /// C03 - Razão social do emitente (OBRIGATÓRIO).
    /// </summary>
    [XmlElement("xNome")]
    public required string RazaoSocial { get; set; }
    
    /// <summary>
    /// C06 - Inscrição Estadual do emitente (OBRIGATÓRIO).
    /// </summary>
    [XmlElement("IE")]
    public required string InscricaoEstadual { get; set; }
    
    /// <summary>
    /// Endereço do emitente (OBRIGATÓRIO).
    /// </summary>
    [XmlElement("enderEmit")]
    public required EnderecoEmitente Endereco { get; set; }
}

// ✅ USO - Compilador força inicialização
var emitente = new Emitente
{
    RazaoSocial = "EMPRESA LTDA",           // ✅ Obrigatório
    InscricaoEstadual = "123456789",        // ✅ Obrigatório
    Endereco = new EnderecoEmitente { }     // ✅ Obrigatório
};

// ❌ ERRO DE COMPILAÇÃO - Faltam propriedades required
var emitente = new Emitente();  // ❌ Erro CS9035
```

### Init-Only Properties - Imutabilidade

Para **objetos de valor** (value objects), usar `init`.

```csharp
// ✅ CORRETO - Init-only para imutabilidade
public class ChaveAcesso
{
    /// <summary>
    /// Chave de acesso de 44 dígitos (imutável após criação).
    /// </summary>
    public required string Valor { get; init; }
    
    /// <summary>
    /// UF de origem (imutável).
    /// </summary>
    public required string UF { get; init; }
    
    /// <summary>
    /// CNPJ do emitente (imutável).
    /// </summary>
    public required string Cnpj { get; init; }
}

// ✅ USO - Pode inicializar, mas não modificar depois
var chave = new ChaveAcesso
{
    Valor = "35240612345678901234550010000000011234567890",
    UF = "SP",
    Cnpj = "12345678901234"
};

// ❌ ERRO DE COMPILAÇÃO - Não pode modificar após criação
chave.Valor = "outra-chave";  // ❌ Erro CS8852
```

### Global Usings - Reduzir Repetição

Criar arquivo `GlobalUsings.cs` em cada projeto com usings comuns.

```csharp
// GlobalUsings.cs (raiz do projeto SeuProjeto.DFe.Core)
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading;
global using System.Threading.Tasks;
global using System.Xml;
global using System.Xml.Serialization;
global using Microsoft.Extensions.Logging;
global using FluentValidation;
global using FluentValidation.Results;

// GlobalUsings.cs (projeto SeuProjeto.DFe.NFe)
global using SeuProjeto.DFe.Core.Abstratos;
global using SeuProjeto.DFe.Core.Excecoes;
global using SeuProjeto.DFe.Core.Modelos;
global using SeuProjeto.DFe.NFe.V4.Enumeracoes;
```

---

## 🌍 Cross-Platform - NÃO NEGOCIÁVEL

### Regra: ZERO Dependências de Windows

**OBRIGATÓRIO**: Código deve rodar em Windows, Linux, macOS e Docker containers.

**Proibido:**
- ❌ `System.Drawing` (Windows GDI+)
- ❌ `System.Windows.Forms`
- ❌ `System.Security.Cryptography.Pkcs` (somente A3, não suportado Linux)
- ❌ APIs Win32 (P/Invoke)
- ❌ Caminhos hard-coded (`C:\Temp`, `\\servidor\pasta`)

**Permitido:**
- ✅ `System.Security.Cryptography` (A1 .pfx funciona cross-platform)
- ✅ `System.Security.Cryptography.Xml` (assinatura XML cross-platform)
- ✅ `Path.Combine()`, `Path.DirectorySeparatorChar`
- ✅ `Environment.SpecialFolder`

```csharp
// ✅ CORRETO - Cross-platform
public class GerenciadorCertificados
{
    /// <summary>
    /// Carrega certificado A1 (.pfx) de forma cross-platform.
    /// </summary>
    public X509Certificate2 CarregarCertificado(string caminhoArquivo, string senha)
    {
        // ✅ X509Certificate2 funciona em todos os SOs
        return new X509Certificate2(
            caminhoArquivo,
            senha,
            X509KeyStorageFlags.Exportable | X509KeyStorageFlags.PersistKeySet
        );
    }
}

// ✅ CORRETO - Caminhos cross-platform
public class GerenciadorArquivos
{
    /// <summary>
    /// Salva XML em diretório temporário (cross-platform).
    /// </summary>
    public async Task SalvarXmlAsync(string nomeArquivo, string conteudo, CancellationToken ct)
    {
        // ✅ Environment.SpecialFolder funciona em todos os SOs
        string diretorioTemp = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);
        string diretorioDFe = Path.Combine(diretorioTemp, "DFe", "XMLs");
        
        // ✅ Path.Combine usa separador correto (\ no Windows, / no Linux)
        string caminhoCompleto = Path.Combine(diretorioDFe, nomeArquivo);
        
        // Criar diretório se não existir
        Directory.CreateDirectory(diretorioDFe);
        
        await File.WriteAllTextAsync(caminhoCompleto, conteudo, ct);
        
        _logger.LogInformation(
            "XML salvo com sucesso. Caminho={Caminho}",
            caminhoCompleto
        );
    }
}

// ❌ INCORRETO - Windows-specific
public class GerenciadorArquivos
{
    public void Salvar(string conteudo)
    {
        // ❌ Caminho hard-coded Windows
        string caminho = @"C:\Temp\XMLs\nfe.xml";
        
        // ❌ Não funciona em Linux/macOS
        File.WriteAllText(caminho, conteudo);
    }
}

// ❌ INCORRETO - Certificado A3 (SmartCard/Token não funciona em containers)
public class GerenciadorCertificados
{
    public X509Certificate2 CarregarA3()
    {
        // ❌ A3 requer drivers específicos de hardware
        // Não funciona em Docker containers
        X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
        store.Open(OpenFlags.ReadOnly);
        // ...
    }
}
```

### Teste Cross-Platform Obrigatório

Antes de cada release, **DEVE** ser testado em:
- ✅ Windows 11 (amd64)
- ✅ Linux Ubuntu 22.04+ (amd64)
- ✅ macOS (arm64 - Apple Silicon)
- ✅ Docker Alpine Linux (container)

---

## ⚠️ Hierarquia de Exceções - Tipagem Forte

### Estrutura de Exceções Customizadas

```csharp
// Base abstrata (nunca lançar diretamente)
namespace SeuProjeto.DFe.Core.Excecoes;

/// <summary>
/// Exceção base abstrata para todos os erros do sistema DFe.
/// NUNCA lançar esta exceção diretamente - usar subclasses específicas.
/// </summary>
public abstract class DFeException : Exception
{
    protected DFeException(string message) : base(message)
    {
    }
    
    protected DFeException(string message, Exception innerException)
        : base(message, innerException)
    {
    }
}

// Exceções específicas
/// <summary>
/// Exceção lançada quando validação de dados falha (FluentValidation ou manual).
/// </summary>
public class ValidationException : DFeException
{
    public IEnumerable<string> Errors { get; }
    
    public ValidationException(string message, IEnumerable<string> errors)
        : base(message)
    {
        Errors = errors;
    }
}

/// <summary>
/// Exceção lançada quando validação contra XSD SEFAZ falha.
/// </summary>
public class SchemaValidationException : ValidationException
{
    public string SchemaName { get; }
    
    public SchemaValidationException(string message, string schemaName, IEnumerable<string> errors)
        : base(message, errors)
    {
        SchemaName = schemaName;
    }
}

/// <summary>
/// Exceção lançada quando comunicação com SEFAZ falha.
/// </summary>
public class SefazCommunicationException : DFeException
{
    public string? UF { get; }
    public int? StatusCode { get; }
    
    public SefazCommunicationException(string message, string uf, int? statusCode = null)
        : base(message)
    {
        UF = uf;
        StatusCode = statusCode;
    }
    
    public SefazCommunicationException(string message, Exception innerException)
        : base(message, innerException)
    {
    }
}

/// <summary>
/// Exceção lançada quando operação com certificado digital falha.
/// </summary>
public class CertificateException : DFeException
{
    public CertificateException(string message) : base(message)
    {
    }
}

/// <summary>
/// Exceção lançada quando assinatura digital falha.
/// </summary>
public class SignatureException : DFeException
{
    public SignatureException(string message) : base(message)
    {
    }
    
    public SignatureException(string message, Exception innerException)
        : base(message, innerException)
    {
    }
}
```

### Uso das Exceções

```csharp
// ✅ CORRETO - Exceção específica com contexto
public async Task<Protocolo> AutorizarNFeAsync(
    NotaFiscalEletronica nfe,
    CancellationToken ct)
{
    // Validação FluentValidation
    EmitenteValidator validador = new();
    ValidationResult resultado = await validador.ValidateAsync(nfe.Emitente, ct);
    
    if (!resultado.IsValid)
    {
        throw new ValidationException(
            "Dados do emitente inválidos",
            resultado.Errors.Select(e => e.ErrorMessage)
        );
    }
    
    // Validação XSD
    ValidationResult validacaoXsd = _validadorXsd.Validar(xml, "nfe_v4.00.xsd");
    if (!validacaoXsd.IsValid)
    {
        throw new SchemaValidationException(
            "XML não conforme com schema SEFAZ",
            "nfe_v4.00.xsd",
            validacaoXsd.Errors
        );
    }
    
    // Comunicação SEFAZ
    try
    {
        return await EnviarParaSefazAsync(xml, ct);
    }
    catch (HttpRequestException ex)
    {
        throw new SefazCommunicationException(
            $"Falha ao comunicar com SEFAZ {nfe.Emitente.Endereco.UF}",
            nfe.Emitente.Endereco.UF,
            (int?)ex.StatusCode
        );
    }
}

// ❌ INCORRETO - Exception genérica sem contexto
public async Task AutorizarAsync(NotaFiscalEletronica nfe)
{
    if (string.IsNullOrEmpty(nfe.Emitente.Cnpj))
    {
        throw new Exception("CNPJ inválido");  // ❌ Genérica
    }
}
```

---

## ⚡ Async/Await - Sempre Assíncrono

### Regra: I/O Sempre Assíncrono

**TODA** operação de I/O (HTTP, arquivo, banco) **DEVE** ser assíncrona.

```csharp
// ✅ CORRETO - Async em I/O
public class ServicoAutorizacaoNFe(
    IHttpClientFactory httpClientFactory,
    ILogger<ServicoAutorizacaoNFe> logger) : IServicoAutorizacaoNFe
{
    /// <summary>
    /// Envia NFe para autorização na SEFAZ (assíncrono).
    /// </summary>
    public async Task<Protocolo> AutorizarAsync(
        NotaFiscalEletronica nfe,
        CancellationToken ct)
    {
        HttpClient httpClient = httpClientFactory.CreateClient("SEFAZ");
        
        // ✅ Async HTTP
        HttpResponseMessage response = await httpClient.PostAsync(
            url,
            conteudo,
            ct
        );
        
        response.EnsureSuccessStatusCode();
        
        // ✅ Async desserialização
        string xmlResposta = await response.Content.ReadAsStringAsync(ct);
        
        // ✅ Async logging (se configurado async sink)
        logger.LogInformation(
            "NFe autorizada. Protocolo={Protocolo}",
            protocolo.Numero
        );
        
        return protocolo;
    }
}

// ❌ INCORRETO - Sync em I/O (bloqueia thread)
public Protocolo Autorizar(NotaFiscalEletronica nfe)
{
    HttpClient httpClient = new();
    
    // ❌ Sync HTTP (bloqueia thread)
    HttpResponseMessage response = httpClient.PostAsync(url, conteudo).Result;
    
    // ❌ .Result ou .Wait() causa deadlock em UI/ASP.NET
    string xmlResposta = response.Content.ReadAsStringAsync().Result;
    
    return protocolo;
}
```

### CancellationToken - OBRIGATÓRIO

**SEMPRE** aceitar `CancellationToken` em métodos assíncronos.

```csharp
// ✅ CORRETO - CancellationToken em toda operação assíncrona
public async Task<Protocolo> AutorizarAsync(
    NotaFiscalEletronica nfe,
    CancellationToken ct)  // ✅ Sempre presente
{
    ct.ThrowIfCancellationRequested();  // Guard clause
    
    XmlDocument xml = nfe.ToXml();
    
    // Propagar CancellationToken para operações internas
    Protocolo protocolo = await EnviarParaSefazAsync(xml, ct);
    
    return protocolo;
}

// ❌ INCORRETO - Sem CancellationToken
public async Task<Protocolo> AutorizarAsync(NotaFiscalEletronica nfe)  // ❌ Falta CT
{
    // Não há como cancelar esta operação externamente
    return await EnviarParaSefazAsync(xml);
}
```

---

## ✅ Validação - FluentValidation em Todas as Camadas

### Regra: Validar na Entrada + Domínio

**Validação em 2 camadas:**
1. **Entrada (API/Boundary)**: FluentValidation para DTOs
2. **Domínio**: FluentValidation para entidades + XSD para XMLs

```csharp
// ✅ CORRETO - Validador com FluentValidation
namespace SeuProjeto.DFe.NFe.V4.Validadores;

/// <summary>
/// Validador do emitente da NFe.
/// Valida CNPJ, Razão Social, Inscrição Estadual e Endereço.
/// </summary>
public class EmitenteValidator : AbstractValidator<Emitente>
{
    public EmitenteValidator()
    {
        // C02 - CNPJ (condicional: CNPJ OU CPF)
        When(e => e.Cnpj != null, () =>
        {
            RuleFor(e => e.Cnpj)
                .NotEmpty()
                .WithMessage("CNPJ do emitente é obrigatório quando CPF não informado")
                .Length(ConstantesNFe.TAMANHO_CNPJ)
                .WithMessage($"CNPJ deve ter exatamente {ConstantesNFe.TAMANHO_CNPJ} dígitos")
                .Must(CnpjValido)
                .WithMessage("CNPJ inválido - dígito verificador incorreto");
        });
        
        // C03 - Razão Social (obrigatório)
        RuleFor(e => e.RazaoSocial)
            .NotEmpty()
            .WithMessage("Razão Social do emitente é obrigatória")
            .MinimumLength(2)
            .WithMessage("Razão Social deve ter no mínimo 2 caracteres")
            .MaximumLength(60)
            .WithMessage("Razão Social deve ter no máximo 60 caracteres");
        
        // C04 - Nome Fantasia (opcional, mas se informado deve ter 1-60 chars)
        When(e => !string.IsNullOrEmpty(e.NomeFantasia), () =>
        {
            RuleFor(e => e.NomeFantasia)
                .MinimumLength(1)
                .MaximumLength(60)
                .WithMessage("Nome Fantasia deve ter entre 1 e 60 caracteres");
        });
        
        // C06 - Inscrição Estadual (obrigatório)
        RuleFor(e => e.InscricaoEstadual)
            .NotEmpty()
            .WithMessage("Inscrição Estadual do emitente é obrigatória")
            .Must((emitente, ie) => InscricaoEstadualValida(ie, emitente.Endereco?.UF))
            .WithMessage("Inscrição Estadual inválida para a UF informada");
        
        // Endereço (obrigatório)
        RuleFor(e => e.Endereco)
            .NotNull()
            .WithMessage("Endereço do emitente é obrigatório")
            .SetValidator(new EnderecoEmitenteValidator());
    }
    
    private bool CnpjValido(string? cnpj)
    {
        if (string.IsNullOrWhiteSpace(cnpj)) return false;
        
        // Implementação do algoritmo de validação de CNPJ
        // ... código de validação
        return true;
    }
    
    private bool InscricaoEstadualValida(string? ie, string? uf)
    {
        if (string.IsNullOrWhiteSpace(ie)) return false;
        if (ie.Equals("ISENTO", StringComparison.OrdinalIgnoreCase)) return true;
        
        // Algoritmo de validação varia por UF
        return uf switch
        {
            "SP" => ValidarIESaoPaulo(ie),
            "SC" => ValidarIESantaCatarina(ie),
            // ... outras UFs
            _ => true  // UF não implementada ainda
        };
    }
    
    private bool ValidarIESaoPaulo(string ie)
    {
        // Implementação específica de SP
        return true;
    }
    
    private bool ValidarIESantaCatarina(string ie)
    {
        // Implementação específica de SC
        return true;
    }
}

// ✅ CORRETO - Uso do validador
public class ServicoNFe(
    ILogger<ServicoNFe> logger) : IServicoNFe
{
    public async Task<Protocolo> CriarNFeAsync(
        NotaFiscalEletronica nfe,
        CancellationToken ct)
    {
        // Validar emitente
        EmitenteValidator validador = new();
        ValidationResult resultado = await validador.ValidateAsync(nfe.Emitente, ct);
        
        if (!resultado.IsValid)
        {
            string erros = string.Join("; ", resultado.Errors.Select(e => e.ErrorMessage));
            
            logger.LogWarning(
                "Validação do emitente falhou. Erros={Erros}",
                erros
            );
            
            throw new ValidationException(
                "Dados do emitente inválidos",
                resultado.Errors.Select(e => e.ErrorMessage)
            );
        }
        
        logger.LogInformation("Emitente validado com sucesso. CNPJ={CNPJ}", nfe.Emitente.Cnpj);
        
        // ... continuar processamento
        return new Protocolo();
    }
}
```

### Validação XSD - Schema SEFAZ

```csharp
/// <summary>
/// Validador de XMLs contra schemas XSD da SEFAZ.
/// </summary>
public class ValidadorXsd(ILogger<ValidadorXsd> logger) : IValidadorXsd
{
    /// <summary>
    /// Valida XML contra XSD embedded resource.
    /// </summary>
    public ValidationResult Validar(string xml, string nomeSchema)
    {
        List<string> erros = new();
        
        try
        {
            XmlDocument doc = new();
            doc.LoadXml(xml);
            
            // Carregar XSD de embedded resource
            XmlSchemaSet schemas = CarregarSchema(nomeSchema);
            
            // Validar
            doc.Schemas = schemas;
            doc.Validate((sender, args) =>
            {
                if (args.Severity == XmlSeverityType.Error)
                {
                    erros.Add($"Linha {args.Exception.LineNumber}: {args.Message}");
                }
            });
            
            if (erros.Any())
            {
                logger.LogWarning(
                    "Validação XSD falhou. Schema={Schema}, Erros={Erros}",
                    nomeSchema,
                    string.Join("; ", erros)
                );
            }
            else
            {
                logger.LogInformation(
                    "XML validado com sucesso contra schema {Schema}",
                    nomeSchema
                );
            }
            
            return new ValidationResult
            {
                IsValid = !erros.Any(),
                Errors = erros
            };
        }
        catch (Exception ex)
        {
            logger.LogError(
                ex,
                "Erro ao validar XML contra schema {Schema}",
                nomeSchema
            );
            
            return new ValidationResult
            {
                IsValid = false,
                Errors = new[] { $"Erro ao carregar XML: {ex.Message}" }
            };
        }
    }
    
    private XmlSchemaSet CarregarSchema(string nomeSchema)
    {
        XmlSchemaSet schemas = new();
        
        // Carregar de embedded resource (cross-platform)
        Assembly assembly = typeof(ValidadorXsd).Assembly;
        string resourceName = $"SeuProjeto.DFe.NFe.V4.Schemas.{nomeSchema}";
        
        using Stream? stream = assembly.GetManifestResourceStream(resourceName);
        if (stream == null)
        {
            throw new FileNotFoundException(
                $"Schema {nomeSchema} não encontrado em embedded resources"
            );
        }
        
        using XmlReader reader = XmlReader.Create(stream);
        schemas.Add(null, reader);
        
        return schemas;
    }
}
```

---

## 🔄 Resiliência - Polly para Retry e Circuit Breaker

### Regra: SEMPRE Implementar Retry em Chamadas SEFAZ

SEFAZ pode ter instabilidades temporárias. **OBRIGATÓRIO** implementar retry com backoff exponencial.

```csharp
// ✅ CORRETO - Políticas Polly configuradas
namespace SeuProjeto.DFe.Infraestrutura.Resiliencia;

/// <summary>
/// Políticas de resiliência para comunicação com SEFAZ.
/// Implementa retry com backoff exponencial e circuit breaker.
/// </summary>
public static class PollyPolicies
{
    /// <summary>
    /// Política de retry com backoff exponencial.
    /// Tenta até 3 vezes com delay de 2s, 4s, 8s.
    /// </summary>
    public static IAsyncPolicy<HttpResponseMessage> RetryPolicy(ILogger logger) =>
        Policy
            .HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .Or<HttpRequestException>()
            .Or<TaskCanceledException>()
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: tentativa => TimeSpan.FromSeconds(Math.Pow(2, tentativa)),
                onRetry: (outcome, timespan, tentativa, context) =>
                {
                    logger.LogWarning(
                        "Tentativa {Tentativa} falhou. Aguardando {Delay}s antes de tentar novamente. Erro={Erro}",
                        tentativa,
                        timespan.TotalSeconds,
                        outcome.Exception?.Message ?? outcome.Result?.StatusCode.ToString()
                    );
                }
            );
    
    /// <summary>
    /// Política de circuit breaker.
    /// Abre circuito após 5 falhas consecutivas, aguarda 30s antes de tentar novamente.
    /// </summary>
    public static IAsyncPolicy<HttpResponseMessage> CircuitBreakerPolicy(ILogger logger) =>
        Policy
            .HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .Or<HttpRequestException>()
            .CircuitBreakerAsync(
                handledEventsAllowedBeforeBreaking: 5,
                durationOfBreak: TimeSpan.FromSeconds(30),
                onBreak: (outcome, duration) =>
                {
                    logger.LogError(
                        "Circuit breaker ABERTO. Serviço indisponível por {Duracao}s",
                        duration.TotalSeconds
                    );
                },
                onReset: () =>
                {
                    logger.LogInformation("Circuit breaker FECHADO. Serviço disponível novamente");
                },
                onHalfOpen: () =>
                {
                    logger.LogInformation("Circuit breaker MEIO-ABERTO. Testando serviço");
                }
            );
    
    /// <summary>
    /// Política de timeout.
    /// Cancela requisição após 30 segundos.
    /// </summary>
    public static IAsyncPolicy<HttpResponseMessage> TimeoutPolicy() =>
        Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(30));
    
    /// <summary>
    /// Política combinada: timeout + retry + circuit breaker.
    /// Aplica na ordem: timeout → retry → circuit breaker.
    /// </summary>
    public static IAsyncPolicy<HttpResponseMessage> CombinedPolicy(ILogger logger) =>
        Policy.WrapAsync(
            CircuitBreakerPolicy(logger),
            RetryPolicy(logger),
            TimeoutPolicy()
        );
}

// ✅ CORRETO - Configuração no Program.cs
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        // Configurar HttpClient com Polly
        builder.Services.AddHttpClient("SEFAZ", client =>
        {
            client.DefaultRequestHeaders.Add("User-Agent", "DFe.NET/1.0");
            client.Timeout = TimeSpan.FromSeconds(30);
        })
        .AddPolicyHandler((services, request) =>
        {
            ILogger logger = services.GetRequiredService<ILogger<Program>>();
            return PollyPolicies.CombinedPolicy(logger);
        });
        
        var app = builder.Build();
        app.Run();
    }
}
```

---

## 📊 Logging Estruturado - ILogger SEMPRE

### Regra: Logar TODAS as Operações Críticas

**OBRIGATÓRIO** logar:
- ✅ Início e fim de autorização/cancelamento/inutilização
- ✅ Erros de comunicação com SEFAZ
- ✅ Erros de validação (FluentValidation e XSD)
- ✅ Tentativas de retry
- ✅ Mudanças de status de documentos
- ❌ **NUNCA** logar: senhas de certificados, chaves privadas, dados sensíveis de clientes

```csharp
// ✅ CORRETO - Logging estruturado
public class ServicoAutorizacaoNFe(
    IHttpClientFactory httpClientFactory,
    ILogger<ServicoAutorizacaoNFe> logger,
    IAssinadorDigital assinadorDigital) : IServicoAutorizacaoNFe
{
    public async Task<Protocolo> AutorizarAsync(
        NotaFiscalEletronica nfe,
        CertificadoDigital certificado,
        CancellationToken ct)
    {
        // ✅ Log início da operação
        logger.LogInformation(
            "Iniciando autorização de NFe. Chave={ChaveAcesso}, UF={UF}, Ambiente={Ambiente}",
            nfe.ChaveAcesso,
            nfe.Emitente.Endereco.UF,
            nfe.Ambiente
        );
        
        try
        {
            // Assinar XML
            logger.LogDebug(
                "Assinando XML da NFe. Chave={ChaveAcesso}, Certificado={CertificadoSerial}",
                nfe.ChaveAcesso,
                certificado.NumeroSerie  // ✅ OK logar número de série (público)
                // ❌ NUNCA logar: certificado.Senha
            );
            
            XmlDocument xmlAssinado = assinadorDigital.Assinar(nfe.ToXml(), certificado);
            
            // Enviar para SEFAZ
            HttpClient httpClient = httpClientFactory.CreateClient("SEFAZ");
            HttpResponseMessage response = await httpClient.PostAsync(url, conteudo, ct);
            
            // ✅ Log sucesso
            logger.LogInformation(
                "NFe autorizada com sucesso. Chave={ChaveAcesso}, Protocolo={Protocolo}, DataAutorizacao={DataAutorizacao}",
                nfe.ChaveAcesso,
                protocolo.Numero,
                protocolo.DataRecebimento
            );
            
            return protocolo;
        }
        catch (SchemaValidationException ex)
        {
            // ✅ Log erro de validação
            logger.LogError(
                ex,
                "Falha na validação XSD da NFe. Chave={ChaveAcesso}, Schema={Schema}, Erros={Erros}",
                nfe.ChaveAcesso,
                ex.SchemaName,
                string.Join("; ", ex.Errors)
            );
            
            throw;
        }
        catch (SefazCommunicationException ex)
        {
            // ✅ Log erro de comunicação
            logger.LogError(
                ex,
                "Falha na comunicação com SEFAZ. Chave={ChaveAcesso}, UF={UF}, StatusCode={StatusCode}",
                nfe.ChaveAcesso,
                ex.UF,
                ex.StatusCode
            );
            
            throw;
        }
        catch (Exception ex)
        {
            // ✅ Log erro genérico
            logger.LogCritical(
                ex,
                "Erro inesperado ao autorizar NFe. Chave={ChaveAcesso}",
                nfe.ChaveAcesso
            );
            
            throw;
        }
    }
}
```

### Níveis de Log

| Nível       | Quando Usar                                                   |
| ----------- | ------------------------------------------------------------- |
| **Trace**   | Detalhes técnicos muito verbosos (debug profundo)            |
| **Debug**   | Informações de diagnóstico úteis em desenvolvimento          |
| **Information** | Fluxo normal da aplicação (início/fim de operações)      |
| **Warning** | Situação anormal mas recuperável (retry, validação falhada)  |
| **Error**   | Erros que impedem operação mas não param aplicação           |
| **Critical**| Erros que podem parar aplicação (banco indisponível, etc)    |

---

## 📝 Auditoria - Registro Completo de Operações Fiscais

### Regra: Auditoria de 7 Anos (Conformidade Fiscal)

**OBRIGATÓRIO** manter auditoria de:
- ✅ Todas as autorizações (sucesso E falha)
- ✅ Cancelamentos
- ✅ Inutilizações
- ✅ Cartas de correção
- ✅ XMLs enviados e recebidos

**Retenção:** Mínimo 7 anos (conformidade fiscal brasileira).

```csharp
/// <summary>
/// Serviço de auditoria para operações fiscais.
/// Registra TODAS as operações conforme legislação (retenção 7 anos).
/// </summary>
public class AuditoriaService(
    IAuditoriaRepositorio repositorio,
    ILogger<AuditoriaService> logger) : IAuditoriaService
{
    /// <summary>
    /// Registra tentativa de autorização de NFe.
    /// </summary>
    public async Task RegistrarAutorizacaoAsync(
        string chaveAcesso,
        string xmlEnviado,
        string? xmlRecebido,
        bool sucesso,
        string? mensagemErro,
        CancellationToken ct)
    {
        RegistroAuditoria registro = new()
        {
            Operacao = "NFe:Autorizacao",
            ChaveAcesso = chaveAcesso,
            XmlEnviado = xmlEnviado,
            XmlRecebido = xmlRecebido,
            Sucesso = sucesso,
            MensagemErro = mensagemErro,
            DataOperacao = DateTime.UtcNow,
            UsuarioId = ObterUsuarioAtual(),
            EnderecoIp = ObterEnderecoIp()
        };
        
        await repositorio.SalvarAsync(registro, ct);
        
        logger.LogInformation(
            "Auditoria registrada. Operacao={Operacao}, Chave={ChaveAcesso}, Sucesso={Sucesso}",
            registro.Operacao,
            chaveAcesso,
            sucesso
        );
    }
    
    /// <summary>
    /// Registra cancelamento de NFe.
    /// </summary>
    public async Task RegistrarCancelamentoAsync(
        string chaveAcesso,
        string justificativa,
        string protocolo,
        bool sucesso,
        CancellationToken ct)
    {
        RegistroAuditoria registro = new()
        {
            Operacao = "NFe:Cancelamento",
            ChaveAcesso = chaveAcesso,
            Justificativa = justificativa,
            Protocolo = protocolo,
            Sucesso = sucesso,
            DataOperacao = DateTime.UtcNow,
            UsuarioId = ObterUsuarioAtual()
        };
        
        await repositorio.SalvarAsync(registro, ct);
    }
}

/// <summary>
/// Modelo de auditoria (deve ser persistido por NO MÍNIMO 7 anos).
/// </summary>
public class RegistroAuditoria
{
    public Guid Id { get; set; }
    public string Operacao { get; set; } = string.Empty;  // NFe:Autorizacao, NFe:Cancelamento, etc
    public string ChaveAcesso { get; set; } = string.Empty;
    public string? XmlEnviado { get; set; }
    public string? XmlRecebido { get; set; }
    public bool Sucesso { get; set; }
    public string? MensagemErro { get; set; }
    public string? Protocolo { get; set; }
    public string? Justificativa { get; set; }
    public DateTime DataOperacao { get; set; }
    public string? UsuarioId { get; set; }
    public string? EnderecoIp { get; set; }
}
```

---

## 🧪 Testes - xUnit + FluentAssertions

### Regra: Cobertura Mínima de 97%

**Meta de cobertura:**
- ✅ **Serviços**: Mínimo 97%
- ✅ **Validadores**: Mínimo 97%
- ✅ **Core (ChaveAcesso, Calculadoras)**: Mínimo 97%
- ⏸️ **DTOs/Modelos**: Não requer testes (apenas propriedades)

### Nomenclatura de Testes

```
MetodoEmTeste_Cenario_ComportamentoEsperado
```

```csharp
// ✅ CORRETO - Estrutura AAA (Arrange, Act, Assert)
namespace SeuProjeto.DFe.NFe.Testes.Validadores;

public class EmitenteValidatorTests
{
    [Fact]
    [Trait("Category", "Unit")]
    public async Task ValidateAsync_CnpjInvalido_DeveRetornarErro()
    {
        // Arrange - Preparar dados
        Emitente emitente = new()
        {
            Cnpj = "12345678901234",  // CNPJ inválido (dígito verificador errado)
            RazaoSocial = "EMPRESA LTDA",
            InscricaoEstadual = "123456789",
            Endereco = new EnderecoEmitente
            {
                UF = "SP",
                Logradouro = "RUA TESTE",
                Numero = "123",
                Bairro = "CENTRO",
                CodigoMunicipio = "3550308",
                NomeMunicipio = "SAO PAULO",
                CEP = "01001000"
            }
        };
        
        EmitenteValidator validador = new();
        
        // Act - Executar ação
        ValidationResult resultado = await validador.ValidateAsync(emitente);
        
        // Assert - Verificar resultado
        resultado.IsValid.Should().BeFalse();
        resultado.Errors.Should().ContainSingle(e => 
            e.ErrorMessage.Contains("CNPJ inválido"));
    }
    
    [Fact]
    [Trait("Category", "Unit")]
    public async Task ValidateAsync_RazaoSocialVazia_DeveRetornarErro()
    {
        // Arrange
        Emitente emitente = new()
        {
            Cnpj = "12345678000195",  // CNPJ válido
            RazaoSocial = "",         // Inválido: vazio
            InscricaoEstadual = "123456789",
            Endereco = new EnderecoEmitente { UF = "SP" }
        };
        
        EmitenteValidator validador = new();
        
        // Act
        ValidationResult resultado = await validador.ValidateAsync(emitente);
        
        // Assert
        resultado.IsValid.Should().BeFalse();
        resultado.Errors.Should().Contain(e => 
            e.PropertyName == nameof(Emitente.RazaoSocial));
    }
    
    [Fact]
    [Trait("Category", "Unit")]
    public async Task ValidateAsync_EmitenteValido_DeveTerSucesso()
    {
        // Arrange
        Emitente emitente = new()
        {
            Cnpj = "12345678000195",
            RazaoSocial = "EMPRESA TESTE LTDA",
            NomeFantasia = "TESTE",
            InscricaoEstadual = "123456789012",
            Endereco = new EnderecoEmitente
            {
                UF = "SP",
                Logradouro = "RUA TESTE",
                Numero = "123",
                Bairro = "CENTRO",
                CodigoMunicipio = "3550308",
                NomeMunicipio = "SAO PAULO",
                CEP = "01001000"
            }
        };
        
        EmitenteValidator validador = new();
        
        // Act
        ValidationResult resultado = await validador.ValidateAsync(emitente);
        
        // Assert
        resultado.IsValid.Should().BeTrue();
        resultado.Errors.Should().BeEmpty();
    }
}

// ✅ CORRETO - Teste de integração (Mock HTTP)
[Trait("Category", "Integration")]
public class ServicoAutorizacaoNFeTests
{
    [Fact]
    public async Task AutorizarAsync_SefazRetorna200_DeveRetornarProtocolo()
    {
        // Arrange
        Mock<IHttpClientFactory> mockFactory = new();
        Mock<ILogger<ServicoAutorizacaoNFe>> mockLogger = new();
        Mock<IAssinadorDigital> mockAssinador = new();
        Mock<IValidadorXsd> mockValidador = new();
        Mock<IProviderEnderecosSefaz> mockProvider = new();
        
        // Configurar mock para retornar sucesso
        Mock<HttpMessageHandler> mockHandler = new();
        mockHandler.Protected()
            .Setup<Task<HttpResponseMessage>>(
                "SendAsync",
                ItExpr.IsAny<HttpRequestMessage>(),
                ItExpr.IsAny<CancellationToken>()
            )
            .ReturnsAsync(new HttpResponseMessage
            {
                StatusCode = HttpStatusCode.OK,
                Content = new StringContent("<retorno>...</retorno>")
            });
        
        HttpClient httpClient = new(mockHandler.Object);
        mockFactory.Setup(f => f.CreateClient("SEFAZ")).Returns(httpClient);
        
        ServicoAutorizacaoNFe servico = new(
            mockFactory.Object,
            mockLogger.Object,
            mockAssinador.Object,
            mockValidador.Object,
            mockProvider.Object
        );
        
        NotaFiscalEletronica nfe = CriarNFeValida();
        CertificadoDigital certificado = CriarCertificadoMock();
        
        // Act
        Protocolo protocolo = await servico.AutorizarAsync(nfe, certificado, CancellationToken.None);
        
        // Assert
        protocolo.Should().NotBeNull();
        protocolo.Numero.Should().NotBeNullOrEmpty();
    }
}
```

---

## ✅ Checklist Pré-Commit

Antes de fazer commit, verificar:

### ✅ Código

- [ ] **100% Português** (classes, métodos, variáveis, comentários, commits)
- [ ] **Uma classe por arquivo** (sem exceções)
- [ ] **Documentação XML completa** em todos os membros públicos (inclui tags `<summary>`, `<param>`, `<returns>`, `<exception>`)
- [ ] **Guard clauses** no início de todos os métodos (early return)
- [ ] **Switch expressions** ao invés de múltiplos if/else
- [ ] **Primary constructors** em serviços com DI
- [ ] **File-scoped namespaces** (sem chaves de namespace)
- [ ] **Required members** em DTOs obrigatórios
- [ ] **Tipos explícitos** (decimal, int, bool) em cálculos fiscais/monetários
- [ ] **CancellationToken** em todos os métodos async

### ✅ Valores e Datas

- [ ] **decimal** para TODOS os valores monetários/fiscais (NUNCA double/float)
- [ ] **ZERO arredondamento** em cálculos (sem Math.Round)
- [ ] **DateTime.UtcNow** para persistência (NUNCA DateTime.Now)
- [ ] **TimeZoneConfiguracao.ParaBrasil()** para exibição UTC-3

### ✅ Cross-Platform

- [ ] **ZERO dependências Windows** (sem System.Drawing, System.Windows, Win32)
- [ ] **Path.Combine()** para todos os caminhos de arquivo
- [ ] **Environment.SpecialFolder** ao invés de caminhos hard-coded
- [ ] **Certificados A1 (.pfx)** apenas (sem A3/SmartCard)

### ✅ Validação e Resiliência

- [ ] **FluentValidation** em todas as entidades/DTOs
- [ ] **Validação XSD** antes de enviar para SEFAZ
- [ ] **Polly retry policy** configurada em HttpClient SEFAZ
- [ ] **Circuit breaker** configurado para alta disponibilidade

### ✅ Logging e Auditoria

- [ ] **ILogger** injetado em todos os serviços
- [ ] **Logging estruturado** (usar template strings, não interpolação)
- [ ] **NUNCA logar** senhas de certificados ou dados sensíveis
- [ ] **Auditoria** de todas as operações fiscais (autorização, cancelamento, inutilização)

### ✅ Testes

- [ ] **Nomenclatura**: `MetodoEmTeste_Cenario_ComportamentoEsperado`
- [ ] **Padrão AAA** (Arrange, Act, Assert)
- [ ] **[Trait("Category")]** para categorizar (Unit, Integration)
- [ ] **FluentAssertions** para assertions legíveis
- [ ] **Cobertura mínima** atingida (97% em todos os módulos)

### ✅ Commit

- [ ] **Conventional Commits** em português
- [ ] **Mensagem robusta e descritiva** (não seja minimalista, mas também não escreva um README)
- [ ] **Organização por categorias** quando commit abrange múltiplas áreas
- [ ] **Contexto suficiente** para entender mudanças sem comparar com commit anterior

#### Formato de Mensagem de Commit

**Estrutura:**
```
<tipo>: <título curto e objetivo>

<resumo opcional em 1-2 linhas do que foi realizado>

<SEÇÃO 1 (quando aplicável)>:
- item 1: descrição concisa mas completa
- item 2: descrição concisa mas completa

<SEÇÃO 2 (quando aplicável)>:
- item 1: descrição concisa mas completa

<Status/Observações finais (quando aplicável)>
```

**Tipos de commit:**
- `feat`: nova funcionalidade
- `fix`: correção de bug
- `refactor`: refatoração sem alterar comportamento
- `docs`: alterações em documentação
- `test`: adição ou correção de testes
- `chore`: configurações, estrutura, dependências
- `perf`: melhorias de performance
- `style`: formatação, linting (sem lógica)
- `ci`: alterações em CI/CD

#### Exemplos de Commits Robustos

**Exemplo 1 - Commit Estrutural:**
```
chore: estrutura inicial do projeto DFe.NET com padrões e configurações

Estabelece fundação completa da biblioteca - sistema de documentos fiscais eletrônicos.

DOCUMENTAÇÃO:
- README.md: visão geral da biblioteca, documentos suportados (NFe, CTe, MDFe, 
  NFCom, NFSe), stack tecnológico .NET 10, arquitetura por namespace versionado,
  princípios fundamentais (cross-platform, português, XSD como verdade, zero 
  arredondamento, UTC-3)
- PADRONIZACAO.md (2766 linhas): padrões obrigatórios incluindo nomenclatura híbrida
  (C# + XML SEFAZ), valores monetários decimal sem arredondamento, timezone UTC-3,
  recursos .NET 10 (primary constructors, file-scoped namespaces, required members),
  cross-platform absoluto, validação FluentValidation + XSD, resiliência Polly,
  auditoria 7 anos, cobertura 97%
- ARQUITETURA.md: decisões sobre namespace por versão, DTOs manuais vs auto-gerados,
  embedded XSD resources, separação NFe/CTe/MDFe/NFCom/NFSe

VSCODE:
- settings.json: configuração completa (C# formatting Allman, 4 espaços, analyzers)
- extensions.json: extensões recomendadas (C#, EditorConfig, XML, Git, Docker)
- launch.json: configurações debug (biblioteca, testes, samples)
- tasks.json: tasks build/test/pack/publish

GITHUB:
- workflows/ci.yml: pipeline CI (build .NET 10, testes xUnit, cobertura 97%,
  análise CodeQL, validação XSD embedded resources, certificação cross-platform
  em Windows/Linux/macOS)
- ISSUE_TEMPLATE: templates para bug fiscal, feature, schema SEFAZ atualizado
- PULL_REQUEST_TEMPLATE: checklist PADRONIZACAO.md compliance
- dependabot.yml: atualização .NET packages e GitHub Actions

INFRAESTRUTURA:
- .gitignore: .NET artifacts, certificados .pfx (NUNCA commitar), XMLs temporários,
  logs auditoria, bin/obj, IDE configs
- .editorconfig: Allman braces, 4 espaços, file-scoped namespaces, nullable enabled

Status: estrutura base pronta, desenvolvimento de NFe V4 pode iniciar.
```

**Exemplo 2 - Feature Completa:**
```
feat: implementa autorização de NFe V4 com assinatura digital e validação XSD

Adiciona funcionalidade completa de autorização de Nota Fiscal Eletrônica versão 4.00
junto à SEFAZ, incluindo assinatura digital, validações e resiliência.

MODELOS (NFe.V4.Modelos):
- NotaFiscalEletronica.cs: estrutura completa <NFe> com 15 propriedades mapeadas
- Emitente.cs: C01-C19 conforme manual SEFAZ (CNPJ/CPF, IE, endereço)
- Destinatario.cs: E01-E19 com validação condicional (CNPJ ou CPF)
- Produto.cs: I01-I80 incluindo tributação ICMS/IPI/PIS/COFINS
- Total.cs: W01-W23 valores totais e tributos

VALIDADORES (NFe.V4.Validadores):
- ValidadorEmitente.cs: FluentValidation com 12 regras (CNPJ válido, IE por UF,
  endereço completo, CEP formato correto)
- ValidadorDestinatario.cs: validação condicional CNPJ/CPF + IE quando aplicável
- ValidadorProduto.cs: NCM obrigatório, CEST quando aplicável, valores > 0

SERVIÇOS (NFe.V4.Servicos):
- ServicoAutorizacaoNFe.cs: fluxo completo (validar → assinar → enviar → processar)
  com retry Polly (3 tentativas, backoff exponencial 2/4/8s), circuit breaker
  (5 falhas = 30s pausa), timeout 30s
- ServicoConsultaNFe.cs: consulta protocolo por chave acesso
- ServicoConsultaStatusSefaz.cs: verifica disponibilidade SEFAZ por UF

INFRAESTRUTURA:
- AssinadorDigital.cs: assinatura SHA-256 com certificado A1 (.pfx), suporta
  múltiplas tags <Signature>, cross-platform via System.Security.Cryptography.Xml
- ValidadorXsd.cs: validação contra nfe_v4.00.xsd embedded resource, retorna
  lista detalhada de erros com linha/coluna
- ProvedorEnderecosSefaz.cs: URLs SEFAZ por UF e ambiente (produção/homologação)
  para 27 UFs, fallback SVRS quando UF não possui webservice próprio

TESTES (NFe.Testes.V4):
- ValidadorEmitenteTestes.cs: 15 cenários cobrindo validações (97% coverage)
- ServicoAutorizacaoNFeTestes.cs: mocks HttpClient, testa retry/timeout/circuit breaker
- AssinadorDigitalTestes.cs: valida assinatura com certificado teste, cross-platform

SCHEMAS (NFe.V4.Schemas - embedded resources):
- nfe_v4.00.xsd, tiposBasico_v4.00.xsd, xmldsig-core-schema_v1.01.xsd

AUDITORIA:
- Logs estruturados: início/sucesso/falha autorização com chave, protocolo, timestamp
- Registro auditoria: XML enviado + recebido persistidos (retenção 7 anos)

Testes: 142 testes passando, cobertura 97.3% (ValidadorEmitente 98%, ServicoAutorizacao 96%)
Compatibilidade: testado Windows 11, Ubuntu 22.04, macOS 14 (Apple Silicon)
Performance: autorização média 850ms (rede SEFAZ SP homologação)
```

**Exemplo 3 - Fix Crítico:**
```
fix: corrige cálculo de dígito verificador em chaves de acesso para UF 91/92

BUG: chaves de acesso geradas para ambiente homologação (UF 91/92) resultavam
em dígito verificador incorreto, causando Rejeição 215 pela SEFAZ.

CAUSA RAIZ:
- CalculadoraDigitoVerificador.cs linha 43: algoritmo módulo 11 não tratava
  corretamente códigos UF sintéticos (91=todas UFs homologação, 92=SVC-AN/SVC-RS)
- Estava aplicando peso 2-9 apenas nos primeiros 42 dígitos, ignorando que
  para UF 91/92 o código UF também deve entrar no cálculo

CORREÇÃO:
- CalculadoraDigitoVerificador.cs: algoritmo atualizado para incluir todos os
  43 dígitos no cálculo (incluindo UF sintética)
- Adicionada validação explícita: se UF >= 91, aplicar mesmo algoritmo
- ValidadorChaveAcesso.cs: validação adicional comparando resultado com tabela
  oficial SEFAZ para UFs sintéticas

TESTES ADICIONADOS:
- CalculadoraDigitoVerificadorTestes.cs: 8 novos casos para UF 91 e 92
- Casos de teste com chaves reais de homologação (fornecidas SEFAZ)
- Teste de regressão: UFs normais (11-53) continuam funcionando corretamente

VALIDAÇÃO:
- Testado em homologação: 50 NFes autorizadas com sucesso (25 UF 91, 25 UF 92)
- Zero rejeições 215 após correção
- Cobertura CalculadoraDigitoVerificador: 97% → 99%

Relacionado: issue #127 - múltiplos relatos de Rejeição 215 em homologação
```

**Exemplo 4 - Refactor:**
```
refactor: reorganiza validadores NFe por grupos semânticos e extrai regras comuns

Move validadores de estrutura flat para organização hierárquica por domínio fiscal,
extraindo validações reutilizáveis para base abstrata.

ANTES (NFe.V4.Validadores/):
- 23 arquivos validadores no mesmo diretório
- Duplicação: validação CNPJ repetida 8 vezes
- Duplicação: validação CEP repetida 5 vezes
- Sem herança: cada validador implementa tudo do zero

DEPOIS (NFe.V4.Validadores/):
- Base/
  - ValidadorBase.cs: regras comuns (CNPJ, CPF, IE, CEP, telefone, email)
  - ValidadorEnderecoBase.cs: validações endereço (logradouro, bairro, município)
- Identificacao/
  - ValidadorEmitente.cs: herda ValidadorBase + ValidadorEnderecoBase
  - ValidadorDestinatario.cs: herda ValidadorBase + ValidadorEnderecoBase
- Produtos/
  - ValidadorProduto.cs: validações item (NCM, CEST, unidade, valores)
  - ValidadorTributos.cs: validações fiscais (ICMS, IPI, PIS, COFINS)
- Totalizacao/
  - ValidadorTotal.cs: soma produtos = total NFe
  - ValidadorTributosTotais.cs: soma tributos = campos totalizadores
- Transporte/
  - ValidadorTransportadora.cs
  - ValidadorVeiculo.cs

VALIDAÇÕES EXTRAÍDAS (ValidadorBase.cs):
- ValidarCnpj(): algoritmo módulo 11 com cache (melhoria performance 40%)
- ValidarCpf(): algoritmo módulo 11 com cache
- ValidarInscricaoEstadual(): 27 algoritmos por UF centralizados
- ValidarCep(): formato + dígito verificador
- ValidarTelefone(): DDD + número, aceita fixo/celular
- ValidarEmail(): regex RFC 5322 compliant

BENEFÍCIOS:
- Redução duplicação: 856 linhas → 234 linhas (72% redução)
- Performance: cache validações repetidas (CNPJ emitente validado 1 vez)
- Manutenção: atualização algoritmo IE afeta 1 lugar, não 8
- Testes: ValidadorBaseTestes cobre validações comuns (97% coverage)

COMPATIBILIDADE:
- API pública não alterada: ValidadorEmitente ainda funciona igual
- Testes existentes: 142 testes passando sem alteração
- Breaking change: NENHUM (apenas refactor interno)
```

**Exemplo 5 - Simples mas Objetivo:**
```
docs: adiciona exemplos de uso para autorização NFe no README

Inclui 3 exemplos práticos step-by-step:
1. Autorização NFe básica (código mínimo funcional)
2. Autorização com tratamento completo de erros (validação, comunicação, rejeição)
3. Fluxo completo: criação → validação → assinatura → autorização → consulta

Cada exemplo inclui comentários explicativos e links para seções relevantes
da documentação SEFAZ.
```

#### ❌ Exemplos de Commits Inadequados

**Muito vago:**
```
❌ fix: corrige bug
❌ feat: adiciona validação
❌ refactor: melhora código
```

**Muito minimalista (falta contexto):**
```
❌ feat: adiciona ServicoAutorizacaoNFe.cs

O QUE esse serviço faz? Quais validações? Usa Polly? Testa o quê?
```

**Muito verboso (virou README):**
```
❌ feat: implementa autorização NFe

[... 300 linhas explicando cada linha de código alterada ...]
[... incluindo trechos de código completos ...]
[... documentação repetida do que já está no README ...]
```

**Em inglês:**
```
❌ feat: add NFe authorization service
```

---

## 📚 Referências Técnicas

### Documentação Oficial SEFAZ

- [Manual de Integração Contribuinte NFe v4.00](http://www.nfe.fazenda.gov.br/portal/principal.aspx)
- [Schemas XSD NFe (Pacote 010b v1.30)](http://www.nfe.fazenda.gov.br/portal/exibirArquivo.aspx?conteudo=9dtVu58wN+Y=)
- [Manual de Integração CTe v4.00](http://www.cte.fazenda.gov.br/)
- [Manual de Integração MDFe v3.00](https://mdfe.fazenda.gov.br/)
- [Padrão Nacional NFS-e v1.02](https://www.gov.br/nfse/)

### Tecnologias

- [.NET 10 Documentation](https://learn.microsoft.com/pt-br/dotnet/)
- [FluentValidation](https://docs.fluentvalidation.net/)
- [Polly](https://github.com/App-vNext/Polly)
- [xUnit](https://xunit.net/)
- [FluentAssertions](https://fluentassertions.com/)

---

**FIM DO DOCUMENTO DE PADRONIZAÇÃO**

Este documento deve ser revisado e atualizado sempre que novas decisões arquiteturais forem tomadas ou novas versões de layouts SEFAZ forem lançadas.

Última atualização: 21/01/2026
Versão: 1.0.0