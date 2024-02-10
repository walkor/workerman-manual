# Como integrar com outros frameworks

**Pergunta:**

Como integrar com outros frameworks MVC (como thinkPHP, Yii, etc)?

**Resposta:**

![Workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

É **recomendado** integrar com outros frameworks MVC da seguinte forma (usando ThinkPHP como exemplo):

1. ThinkPHP e Workerman são dois sistemas independentes, com implantações separadas (podem ser implantados em servidores diferentes) e não interferem um no outro.

2. ThinkPHP fornece páginas da web para renderização no navegador usando o protocolo HTTP.

3. As páginas fornecidas pelo ThinkPHP iniciam uma conexão websocket, conectando-se ao Workerman.

4. Após a conexão, envia-se um pacote de dados para o Workerman (incluindo nome de usuário, senha ou algum tipo de token) para validar a conexão websocket pertencente a qual usuário.

5. Somente quando o ThinkPHP precisa enviar dados para o navegador, é chamada a interface de socket do Workerman para enviar os dados.

6. As demais solicitações continuam sendo tratadas no ThinkPHP usando o método HTTP original.

**Conclusão:**

O Workerman é usado como um canal para enviar dados para o navegador, sendo a interface do Workerman chamada apenas quando é necessário enviar dados para o navegador. A lógica de negócios é totalmente concluída no ThinkPHP.

Para saber como o ThinkPHP chama a interface de socket do Workerman para enviar dados, consulte a seção [FAQ - Pushing in Other Project](push-in-other-project.md).

**O ThinkPHP já suporta oficialmente o workerman, consulte o [manual do ThinkPHP5](https://www.kancloud.cn/manual/thinkphp5/235128)**.
