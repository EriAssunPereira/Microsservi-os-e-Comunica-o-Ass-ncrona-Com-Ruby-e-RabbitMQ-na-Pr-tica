# Microsserviços-e-Comunicação-Assíncrona-Com-Ruby-e-RabbitMQ-na-Prática

Vamos detalhar como poderemos criar um projeto de microsserviços utilizando Ruby e RabbitMQ, focando na comunicação assíncrona. Vou dividir em módulos para facilitar o entendimento.

### Módulo 1: Configuração Básica

1. **Instalação do RabbitMQ**: Primeiro, certifique-se de ter o RabbitMQ instalado localmente ou em um servidor. Você pode baixá-lo e instalá-lo seguindo as instruções do site oficial.

2. **Configuração do Projeto Ruby**: Inicie um novo projeto Ruby usando o framework desejado (como Rails ou Sinatra).

   ```bash
   # Exemplo com Rails
   rails new microservices_project --skip-active-record --skip-test
   cd microservices_project
   ```

### Módulo 2: Configuração do RabbitMQ

1. **Instalação da Gem do RabbitMQ**: Adicione a gem `bunny` ao seu `Gemfile` e execute o `bundle install` para instalar as dependências.

   ```ruby
   # Gemfile
   gem 'bunny'
   ```

   ```bash
   bundle install
   ```

2. **Configuração da Conexão com o RabbitMQ**: Configure a conexão com o RabbitMQ em um arquivo de inicialização (`config/initializers/rabbitmq.rb`).

   ```ruby
   # config/initializers/rabbitmq.rb
   require 'bunny'

   $rabbitmq_connection = Bunny.new(
     host: 'localhost',
     port: 5672,
     user: 'guest',
     password: 'guest'
   )

   $rabbitmq_connection.start
   ```

### Módulo 3: Criando Microsserviços

1. **Criando um Produtor**: Vamos criar um microsserviço que atua como produtor, enviando mensagens para uma fila no RabbitMQ.

   ```ruby
   # app/services/message_producer_service.rb
   class MessageProducerService
     def initialize
       @channel = $rabbitmq_connection.create_channel
     end

     def publish_message(message)
       exchange = @channel.default_exchange
       exchange.publish(message, routing_key: 'messages')
     end
   end
   ```

2. **Criando um Consumidor**: Agora, vamos criar um microsserviço que atua como consumidor, recebendo mensagens da fila no RabbitMQ.

   ```ruby
   # app/services/message_consumer_service.rb
   class MessageConsumerService
     def initialize
       @channel = $rabbitmq_connection.create_channel
       @queue = @channel.queue('messages')
     end

     def start_consuming
       @queue.subscribe(block: true) do |delivery_info, properties, body|
         process_message(body)
       end
     end

     def process_message(body)
       puts "Mensagem recebida: #{body}"
       # Lógica para processar a mensagem aqui
     end
   end
   ```

### Módulo 4: Exemplo de Uso

1. **Utilizando os Serviços**: Vamos agora utilizar os serviços que criamos para enviar e receber mensagens.

   ```ruby
   # Exemplo de uso no console ou em um controlador
   producer = MessageProducerService.new
   producer.publish_message("Olá, RabbitMQ!")

   # Em outro terminal ou processo
   consumer = MessageConsumerService.new
   consumer.start_consuming
   ```

### Módulo 5: Considerações Finais

1. **Execução e Escalabilidade**: Lembre-se de que microsserviços podem ser executados independentemente e escalados conforme necessário, permitindo uma arquitetura flexível e robusta.

2. **Monitoramento e Gerenciamento**: Considere implementar ferramentas de monitoramento e gerenciamento para acompanhar o desempenho e a saúde dos seus microsserviços.

Este projeto básico demonstra como criar microsserviços em Ruby utilizando RabbitMQ para comunicação assíncrona. Podemos expandir isso adicionando mais microsserviços, implementando filas específicas para diferentes tipos de mensagens e integrando com outras tecnologias conforme necessário para seu projeto.
