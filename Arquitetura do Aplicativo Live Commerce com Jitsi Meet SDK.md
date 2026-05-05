# Arquitetura do Aplicativo Live Commerce com Jitsi Meet SDK

**Autor:** Manus AI  
**Data:** Maio de 2026  
**Versão:** 1.0.0

---

## 1. Visão Geral

O **Live Commerce** é um aplicativo Android nativo que integra o Jitsi Meet SDK para transmissões de vídeo ao vivo combinadas com uma interface customizada que permite interações comerciais em tempo real. A plataforma permite que apresentadores transmitam produtos ao vivo enquanto os espectadores podem visualizar, fazer perguntas via chat, reagir com emojis e realizar compras sem sair da transmissão.

O diferencial arquitetural reside na sobreposição de componentes de UI customizados sobre o `JitsiMeetView`, criando uma experiência imersiva onde o vídeo, chat, produtos e botões de compra coexistem de forma harmoniosa. A arquitetura segue os princípios de **Clean Architecture** com separação clara entre camadas de apresentação, domínio e dados, garantindo manutenibilidade, testabilidade e escalabilidade.

---

## 2. Princípios Arquiteturais

A estrutura do Live Commerce é construída sobre cinco pilares fundamentais que guiam todas as decisões de design:

**Separação de Preocupações:** Cada componente do sistema possui uma responsabilidade única e bem definida. A UI não contém lógica de negócio, os casos de uso não conhecem detalhes de implementação de dados, e os repositórios abstraem completamente as fontes de dados. Essa separação permite que mudanças em uma camada não afete as demais.

**Modularidade:** O projeto está dividido em cinco módulos Gradle independentes (`:app`, `:domain`, `:data`, `:jitsi-wrapper`, `:common-ui`), cada um com suas próprias responsabilidades. Essa estrutura modular facilita o desenvolvimento paralelo, testes isolados e reutilização de componentes em outros projetos.

**Reusabilidade:** Componentes genéricos como `ProductCard`, `ChatOverlay` e `ReactionEmoji` são projetados para serem reutilizados em diferentes contextos da aplicação. O módulo `:common-ui` centraliza todos esses componentes, evitando duplicação de código.

**Testabilidade:** A arquitetura foi projetada desde o início para facilitar testes unitários, de integração e de UI. Interfaces de repositório, injeção de dependência e separação clara de responsabilidades permitem que cada camada seja testada isoladamente.

**Performance:** Otimizações são consideradas desde o design inicial, especialmente para lidar com transmissão de vídeo ao vivo, múltiplos participantes, chat em tempo real e atualização frequente de dados de produtos. Qualidade adaptativa de vídeo, cache inteligente e gerenciamento eficiente de memória são considerações constantes.

---

## 3. Camadas da Arquitetura

A arquitetura segue o padrão **Clean Architecture** com três camadas principais, cada uma com responsabilidades bem definidas:

### 3.1 Camada de Apresentação (UI/UX)

A camada de apresentação é responsável por toda interação com o usuário e renderização da interface. Desenvolvida em **Kotlin** com **Jetpack Compose**, oferece uma UI moderna e reativa que responde imediatamente às ações do usuário.

**Componentes Principais:**

- **Activities:** `MainActivity` (entrada do app) e `LiveStreamActivity` (transmissão ao vivo)
- **ViewModels:** Gerenciam o estado da UI e orquestram interações com a camada de domínio
- **Composables:** Componentes reutilizáveis de UI construídos com Jetpack Compose
- **JitsiMeetView:** Integrado em um `ConstraintLayout` para permitir sobreposição de elementos customizados
- **Overlays Customizados:** Cards de produto, chat em tempo real, reações visuais, botões de compra

A estratégia de UI é sobrepor elementos customizados sobre o `JitsiMeetView` usando `ConstraintLayout` ou `FrameLayout`. Isso permite que o vídeo da conferência Jitsi permaneça como elemento central enquanto componentes comerciais flutuam sobre ele.

### 3.2 Camada de Domínio (Business Logic)

A camada de domínio contém toda a lógica de negócio do aplicativo, completamente independente de frameworks Android ou tecnologias específicas. Essa camada é o coração da aplicação.

**Componentes Principais:**

- **Casos de Uso (Use Cases):** Cada caso de uso representa uma ação específica do negócio (ex: `JoinLiveRoomUseCase`, `SendChatMessageUseCase`, `PurchaseProductUseCase`). Eles orquestram o fluxo de dados entre apresentação e dados.
- **Entidades:** Representam objetos de negócio como `Product`, `User`, `LiveSession`, `ChatMessage`, `Order`, `Reaction`.
- **Interfaces de Repositório:** Definem contratos para acesso a dados sem expor detalhes de implementação.

### 3.3 Camada de Dados (Data)

A camada de dados é responsável pela recuperação e persistência de dados, abstraindo completamente as fontes (APIs remotas, banco de dados local, cache).

**Componentes Principais:**

- **Repositórios:** Implementam as interfaces definidas na camada de domínio, coordenando múltiplas fontes de dados
- **Data Sources Remotas:** APIs REST para produtos, usuários, histórico de compras, sessões de live
- **Data Sources Locais:** Cache em memória, banco de dados SQLite com Room para offline
- **Jitsi Wrapper:** Camada de abstração em torno do Jitsi Meet SDK, expondo uma interface simplificada

---

## 4. Fluxo de Dados

O fluxo de dados segue um padrão **unidirecional** que garante previsibilidade e facilita debugging:

```
Ação do Usuário na UI
        ↓
ViewModel processa evento
        ↓
ViewModel chama Caso de Uso
        ↓
Caso de Uso interage com Repositórios
        ↓
Repositório acessa Data Sources
        ↓
Resultado é mapeado de volta para UI
        ↓
UI é atualizada com novo estado
```

Por exemplo, quando um usuário envia uma mensagem de chat:

1. Usuário digita e toca no botão "Enviar"
2. `ChatViewModel` recebe o evento `onSendMessage(text)`
3. ViewModel chama `SendChatMessageUseCase`
4. Use Case valida a mensagem e chama `ChatRepository.sendMessage()`
5. Repositório envia via API e salva localmente
6. Resultado é retornado como `Flow<Result>`
7. ViewModel atualiza seu estado
8. UI observa o estado e renderiza a nova mensagem

---

## 5. Módulos do Projeto

O projeto é dividido em cinco módulos Gradle, cada um com propósito específico:

| Módulo | Responsabilidade | Dependências |
|--------|-----------------|--------------|
| `:app` | Camada de apresentação, Activities, MainActivity, LiveStreamActivity | domain, data, jitsi-wrapper, common-ui |
| `:domain` | Lógica de negócio, Use Cases, Entidades, Interfaces de Repositório | Nenhuma (puro Kotlin) |
| `:data` | Implementação de Repositórios, Data Sources, APIs, Database | domain |
| `:jitsi-wrapper` | Integração com Jitsi Meet SDK, abstração de eventos | domain |
| `:common-ui` | Componentes reutilizáveis, ProductCard, ChatOverlay, ReactionEmoji | Nenhuma (UI pura) |

---

## 6. Integração com Jitsi Meet SDK

O módulo `:jitsi-wrapper` encapsula toda a complexidade da integração com o Jitsi Meet SDK, expondo uma interface limpa e reativa para o restante da aplicação.

### 6.1 Configuração Inicial

```kotlin
val options = JitsiMeetConferenceOptions.Builder()
    .setServerURL(URL("https://meet.jitsi.example.com"))
    .setRoom(roomName)
    .setFeatureFlag("toolbox.enabled", true)
    .setFeatureFlag("filmstrip.enabled", false)
    .setFeatureFlag("welcomepage.enabled", false)
    .setFeatureFlag("add-people.enabled", false)
    .setFeatureFlag("invite.enabled", false)
    .build()

JitsiMeet.setDefaultConferenceOptions(options)
```

### 6.2 Gerenciamento de Eventos

O wrapper registra listeners para eventos importantes da conferência e os expõe através de `Flow<ConferenceEvent>`:

- `ConferenceJoined` - Usuário entrou na sala
- `ConferenceTerminated` - Conferência encerrada
- `ParticipantJoined` - Novo participante entrou
- `ParticipantLeft` - Participante saiu
- `MessageReceived` - Mensagem de chat recebida
- `ScreenShareToggled` - Compartilhamento de tela ativado/desativado

### 6.3 Feature Flags Customizadas

Feature flags permitem customizar o comportamento e aparência da UI do Jitsi:

| Flag | Descrição | Valor Padrão |
|------|-----------|--------------|
| `toolbox.enabled` | Mostrar barra de ferramentas | true |
| `filmstrip.enabled` | Mostrar lista de participantes | true |
| `welcomepage.enabled` | Mostrar página de boas-vindas | true |
| `add-people.enabled` | Permitir adicionar pessoas | false |
| `invite.enabled` | Permitir enviar convites | false |
| `video-share.enabled` | Compartilhamento de vídeo | true |
| `conference-timer.enabled` | Mostrar duração da conferência | true |

---

## 7. Interface Customizada para Interações Comerciais

A interface customizada é construída sobre o `JitsiMeetView` usando `ConstraintLayout` ou `FrameLayout` para sobreposição de elementos. Os principais componentes incluem:

### 7.1 Cards de Produto

Composables que exibem informações do produto (imagem, nome, preço, descrição, botão de compra). Aparecem de forma contextual durante a live, gerenciados por um `ProductViewModel` que interage com a camada de dados para obter informações atualizadas.

**Características:**
- Animação de entrada suave
- Imagem do produto com cache
- Preço destacado
- Botão de compra rápida
- Indicador de disponibilidade

### 7.2 Chat em Tempo Real

Composable que exibe mensagens de chat dos espectadores e permite envio de mensagens. Integrado a um serviço de backend para persistência, moderação e sincronização em tempo real.

**Características:**
- Scroll automático para mensagens novas
- Suporte a emojis
- Indicador de digitação
- Mensagens do apresentador destacadas
- Moderação de conteúdo

### 7.3 Reações/Emojis

Pequenas animações que espectadores podem enviar para expressar emoções durante a transmissão. As reações flutuam sobre o vídeo e desaparecem após alguns segundos.

**Características:**
- Animações fluidas
- Múltiplos emojis disponíveis (👍, ❤️, 😂, 😮, 👏)
- Contador de reações
- Efeito de partículas

### 7.4 Botão de Compra Rápida

Botão proeminente que direciona o usuário para a página de detalhes do produto ou adiciona o item ao carrinho. Permanece visível durante toda a transmissão.

**Características:**
- Posicionamento fixo
- Animação de pulso
- Feedback visual ao clicar
- Integração com carrinho de compras

---

## 8. Modelos de Dados

### 8.1 Entidades Principais

```kotlin
// Usuário
data class User(
    val id: String,
    val name: String,
    val email: String,
    val role: UserRole, // PRESENTER, VIEWER
    val profileImage: String?
)

// Sessão de Live
data class LiveSession(
    val id: String,
    val title: String,
    val description: String,
    val presenterId: String,
    val startTime: Long,
    val endTime: Long?,
    val status: LiveStatus, // SCHEDULED, LIVE, ENDED
    val participantCount: Int
)

// Produto
data class Product(
    val id: String,
    val name: String,
    val description: String,
    val price: Double,
    val image: String,
    val stock: Int,
    val liveSessionId: String
)

// Mensagem de Chat
data class ChatMessage(
    val id: String,
    val senderId: String,
    val senderName: String,
    val content: String,
    val timestamp: Long,
    val liveSessionId: String
)

// Pedido
data class Order(
    val id: String,
    val userId: String,
    val liveSessionId: String,
    val products: List<OrderItem>,
    val totalPrice: Double,
    val status: OrderStatus, // PENDING, CONFIRMED, SHIPPED, DELIVERED
    val createdAt: Long
)

// Reação
data class Reaction(
    val id: String,
    val userId: String,
    val emoji: String,
    val timestamp: Long,
    val liveSessionId: String
)
```

---

## 9. Casos de Uso Principais

| Caso de Uso | Descrição | Entrada | Saída |
|------------|-----------|---------|-------|
| `JoinLiveRoomUseCase` | Entrar em uma sala de transmissão | roomName, userName | LiveSession |
| `LeaveLiveRoomUseCase` | Sair da transmissão | liveSessionId | Success/Error |
| `SendChatMessageUseCase` | Enviar mensagem de chat | message, liveSessionId | ChatMessage |
| `GetChatMessagesUseCase` | Recuperar histórico de chat | liveSessionId | Flow<List<ChatMessage>> |
| `SendReactionUseCase` | Enviar reação de emoji | emoji, liveSessionId | Reaction |
| `GetProductsUseCase` | Obter produtos da live | liveSessionId | Flow<List<Product>> |
| `PurchaseProductUseCase` | Comprar um produto | productId, quantity | Order |
| `GetOrderHistoryUseCase` | Obter histórico de pedidos | userId | Flow<List<Order>> |
| `GetParticipantsUseCase` | Obter lista de participantes | liveSessionId | Flow<List<User>> |

---

## 10. Considerações de Segurança e Performance

### 10.1 Segurança

**Autenticação e Autorização:** Implementar um sistema robusto de autenticação (OAuth 2.0 ou JWT) para garantir que apenas usuários autorizados possam iniciar transmissões ou realizar compras. Apresentadores devem ter permissões elevadas para controlar a transmissão.

**Comunicação Segura:** Toda comunicação com o backend deve ocorrer via HTTPS com certificados válidos. Tokens de autenticação devem ser armazenados de forma segura no Android Keystore.

**Validação de Dados:** Validar todos os dados de entrada (mensagens de chat, informações de produtos, dados de pagamento) tanto no cliente quanto no servidor para prevenir injeção de código ou manipulação de dados.

**Proteção contra Ataques Comuns:** Implementar proteção contra CSRF, XSS (embora menos relevante em Android), SQL Injection (se usar banco de dados local) e rate limiting para APIs.

### 10.2 Performance

**Qualidade de Vídeo Adaptativa:** O Jitsi Meet SDK já implementa adaptação automática de qualidade baseada na largura de banda disponível. Monitorar a qualidade e informar ao usuário se necessário.

**Monitoramento de Rede e Bateria:** Implementar mecanismos para monitorar uso de rede e bateria. Oferecer opções de qualidade reduzida para dispositivos com bateria baixa ou conexão fraca.

**Cache Inteligente:** Implementar cache em múltiplas camadas - imagens de produtos em cache de disco, dados de sessão em memória, histórico de chat em banco de dados local.

**Otimização de Memória:** Usar `Coil` ou `Glide` para carregamento eficiente de imagens, implementar `ViewModel` com `SavedStateHandle` para preservar estado durante rotações de tela, e usar `Flow` para reatividade sem vazamentos de memória.

---

## 11. Stack Tecnológico

### 11.1 Linguagem e Framework

| Tecnologia | Versão | Propósito |
|-----------|--------|----------|
| Kotlin | 1.9.0+ | Linguagem principal |
| Java | 11+ | Compatibilidade |
| Android SDK | 34 (API 34) | Plataforma |
| Jetpack Compose | 1.5.4+ | Framework de UI |

### 11.2 Comunicação e Dados

| Tecnologia | Versão | Propósito |
|-----------|--------|----------|
| Retrofit | 2.9.0 | Cliente HTTP |
| OkHttp | 3.11.0+ | Gerenciamento de requisições |
| Gson | 2.10.1 | Serialização JSON |
| Coroutines | 1.7.0+ | Programação assíncrona |

### 11.3 Transmissão de Vídeo

| Tecnologia | Versão | Propósito |
|-----------|--------|----------|
| Jitsi Meet SDK | Latest | Plataforma de conferência |
| WebRTC | Integrado | Protocolo de comunicação |
| Feature Flags | Integrado | Customização de UI |

### 11.4 Persistência e Cache

| Tecnologia | Versão | Propósito |
|-----------|--------|----------|
| Room | 2.5.0+ | Banco de dados local |
| DataStore | 1.0.0+ | Preferências seguras |
| Coil | 2.4.0+ | Carregamento de imagens |

### 11.5 Arquitetura e Injeção de Dependência

| Tecnologia | Versão | Propósito |
|-----------|--------|----------|
| Hilt | 2.46+ | Injeção de dependência |
| ViewModel | 2.6.0+ | Gerenciamento de estado |
| LiveData/Flow | Integrado | Reatividade |

---

## 12. Estrutura de Diretórios

```
live-commerce-app/
├── app/
│   ├── src/main/
│   │   ├── java/com/example/livecommerce/
│   │   │   ├── ui/
│   │   │   │   ├── MainActivity.kt
│   │   │   │   ├── LiveStreamActivity.kt
│   │   │   │   ├── viewmodel/
│   │   │   │   └── screen/
│   │   │   └── di/
│   │   │       └── AppModule.kt
│   │   └── AndroidManifest.xml
│   └── build.gradle.kts
├── domain/
│   ├── src/main/java/com/example/livecommerce/
│   │   ├── model/
│   │   ├── repository/
│   │   └── usecase/
│   └── build.gradle.kts
├── data/
│   ├── src/main/java/com/example/livecommerce/
│   │   ├── repository/
│   │   ├── datasource/
│   │   ├── api/
│   │   └── db/
│   └── build.gradle.kts
├── jitsi-wrapper/
│   ├── src/main/java/com/example/livecommerce/
│   │   ├── jitsi/
│   │   └── event/
│   └── build.gradle.kts
├── common-ui/
│   ├── src/main/java/com/example/livecommerce/
│   │   └── component/
│   └── build.gradle.kts
├── build.gradle.kts (raiz)
├── settings.gradle.kts
└── README.md
```

---

## 13. Próximos Passos

Com base nesta arquitetura, os próximos passos para implementação incluem:

1. **Setup do Projeto:** Criar estrutura de módulos Gradle e configurar dependências
2. **Implementação do JitsiWrapper:** Encapsular SDK e expor interface reativa
3. **Modelos de Domínio:** Definir entidades e repositórios
4. **Casos de Uso:** Implementar lógica de negócio
5. **UI com Compose:** Criar Activities e Composables
6. **Integração de API:** Conectar com backend
7. **Testes:** Implementar testes unitários e de integração
8. **Otimizações:** Performance, cache, qualidade de vídeo

---

## Referências

[1] [Jitsi Meet Android SDK Documentation](https://jitsi.github.io/handbook/docs/dev-guide/dev-guide-android-sdk/)  
[2] [Jitsi Meet Feature Flags](https://jitsi.github.io/handbook/docs/dev-guide/mobile-feature-flags/)  
[3] [Android Clean Architecture](https://developer.android.com/jetpack/guide)  
[4] [Jetpack Compose Documentation](https://developer.android.com/jetpack/compose)  
[5] [Kotlin Coroutines](https://kotlinlang.org/docs/coroutines-overview.html)
