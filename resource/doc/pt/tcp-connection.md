# Interfaces provided by the Connection class

Em Workerman, existem dois classes importantes: Worker e Connection.

Cada conexão de cliente corresponde a um objeto Connection, que pode ter callbacks configurados, como onMessage, onClose, etc. Além disso, a classe fornece interfaces para enviar dados para o cliente (send) e fechar a conexão (close), bem como outras interfaces necessárias.

Pode-se dizer que o Worker é um contêiner de escuta, responsável por aceitar conexões de clientes e fornecer conexões encapsuladas como objetos Connection para que os desenvolvedores possam operá-las.
