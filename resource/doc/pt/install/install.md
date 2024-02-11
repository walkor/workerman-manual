# Instruções de Instalação
O Workerman é na realidade um pacote de código PHP. Se o seu ambiente PHP estiver configurado corretamente, você só precisa baixar o código-fonte do Workerman ou o demo para executá-lo.

**Instalação via Composer:**
```sh
composer require workerman/workerman
```

> **Nota**
> Alguns dos espelhos do Composer não estão completos. Use o comando `composer config -g --unset repos.packagist` para remover os espelhos.

# Usuários do Windows (Leitura Obrigatória)

A partir da versão 3.5.3 do Workerman, o sistema suporta tanto Windows quanto Linux. Os usuários do Windows precisam configurar as variáveis de ambiente do PHP.

 ` === As seguintes instruções se aplicam apenas ao ambiente do Workerman do sistema Linux, os usuários do Windows devem ignorar === `

# Verificação do Ambiente do Sistema Linux
No sistema Linux, você pode usar o seguinte script para testar se o ambiente PHP local atende aos requisitos de execução do Workerman.
 `curl -Ss https://www.workerman.net/check | php`

Se o script acima mostrar "ok" em todos os casos, isso significa que os requisitos do Workerman estão satisfeitos e você pode baixar o exemplo do [site oficial](https://www.workerman.net/) e executá-lo.

Caso contrário, siga a documentação abaixo para instalar as extensões ausentes.

(Obs.: O script de verificação não inclui a verificação da extensão "event". Se o número de conexões concorrentes exceder 1024, é necessário instalar a extensão "event" e otimizar o kernel Linux. Para instruções sobre a instalação da extensão, consulte o documento abaixo.)
