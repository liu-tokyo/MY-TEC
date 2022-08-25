# GreenMail使用总结

GreenMail是用来测试用的邮件服务器，它支持SMTP、POP3、IMAP等协议，可以镶嵌在任何JAVA代码之中。

## 1. 配置参数

分客户端和服务器两部分

### 客户端：

- email.protocol=smtps

- email.host=localhost

- email.port=465

- email.username=svba@163.com

- email.password=sxj2233

- email.auth=true

- email.systemEmail=test@211.com

### 服务器端：

- email.protocol=smtp

- email.host=localhost

- email.username=test@gmail.com

- email.password=222

- email.auth=true

- email.systemEmail=system@gmail.com

## 2. 启动邮件服务器

- 代码

  ```java
  GreenMail greenmail = new GreenMail(ServerSetup.SMTP);
  
  greenmail.start();
  ```

  

## 3. GreenMail类代码示例

整理汇总了Java中**com.icegreen.greenmail.util.GreenMail\**类\****的典型用法代码示例。如果您正苦于以下问题：Java GreenMail类的具体用法？Java GreenMail怎么用？Java GreenMail使用的例子？那么恭喜您, 这里精选的类代码示例或许可以为您提供帮助。

- 示例1: testHealthCommand

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @Test
  public void testHealthCommand() {
      int smtpPort = SocketUtils.findAvailableTcpPort();
      ServerSetup setup = new ServerSetup(smtpPort, null, ServerSetup.PROTOCOL_SMTP);
      setup.setServerStartupTimeout(5000);
      GreenMail mailServer = new GreenMail(setup);
      mailServer.start();
      ((JavaMailSenderImpl) mailSender).setPort(smtpPort);
      sshCallShell((is, os) -> {
          write(os, "health");
          verifyResponse(is, "{\r\n  \"status\" : \"UP\"");
          mailServer.stop();
      });
  }
  ```

  开发者ID:anand1st，项目名称:sshd-shell-spring-boot，代码行数:15，代码来源:[SshdShellAutoConfigurationTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fanand1st%2Fsshd-shell-spring-boot)

- 示例2: setUp

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @Before
  public void setUp() throws Exception {
      String mailServerHost = "127.0.0.1";
      int mailServerPort = findAvailableTcpPort();
  
      log.warn("Port selected: {}", mailServerPort);
      greenMail = new GreenMail(
          new ServerSetup(mailServerPort, mailServerHost, PROTOCOL_SMTP)
      );
  
      JavaMailSenderImpl sender = new JavaMailSenderImpl();
      sender.setHost(mailServerHost);
      sender.setPort(mailServerPort);
  
      htmlEmailNotificationService = new HtmlEmailNotificationService("[email protected]", sender);
      greenMail.start();
  }
  ```

  开发者ID:AppDirect，项目名称:service-integration-sdk，代码行数:18，代码来源:[HtmlEmailNotificationServiceIntegrationTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FAppDirect%2Fservice-integration-sdk)

- 示例3: setup

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @Before
  public void setup() throws Exception
  {
    node = new SmtpOutputOperator();
    greenMail = new GreenMail(ServerSetupTest.ALL);
    greenMail.start();
    node.setFrom(from);
    node.setContent(content);
    node.setSmtpHost("127.0.0.1");
    node.setSmtpPort(ServerSetupTest.getPortOffset() + ServerSetup.SMTP.getPort());
    node.setSmtpUserName(from);
    node.setSmtpPassword("<password>");
    //node.setUseSsl(true);
    node.setSubject(subject);
    data = new HashMap<String, String>();
    data.put("alertkey", "alertvalue");
  
  }
  ```

  开发者ID:apache，项目名称:apex-malhar，代码行数:19，代码来源:[SmtpOutputOperatorTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fapache%2Fapex-malhar)

- 示例4: setUp

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @Before
  public void setUp() throws Exception {
  
    OnConsoleStatusListener.addNewInstanceToContext(loggerContext);
    MDC.clear();
    ServerSetup serverSetup = new ServerSetup(port, "localhost",
            ServerSetup.PROTOCOL_SMTP);
    greenMailServer = new GreenMail(serverSetup);
    greenMailServer.start();
    // give the server a head start
    if (EnvUtilForTests.isRunningOnSlowJenkins()) {
      Thread.sleep(2000);
    } else {
      Thread.sleep(50);
    }
  }
  ```

  开发者ID:cscfa，项目名称:bartleby，代码行数:17，代码来源:[SMTPAppender_GreenTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fcscfa%2Fbartleby)

- 示例5: setUp

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @Before
  public void setUp() throws Exception {
  
      String username = settings.getString(Mailer.Setting.mail_username, null);
      String password = settings.getString(Mailer.Setting.mail_password, null);
      int port = settings.getInteger(Mailer.Setting.mail_port, 0);
      String systemAddress = settings.getString(Mailer.Setting.mail_systemAddress, null);
  
      // start the test smtp server
      server = new GreenMail(new ServerSetup(port, null, ServerSetup.PROTOCOL_SMTP));
      server.setUser(systemAddress, username, password);
      server.start();
  
      // start the mailer service
      mailer.start();
  }
  ```

  开发者ID:gitblit，项目名称:fathom，代码行数:17，代码来源:[MailerTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fgitblit%2Ffathom)

- 示例6: startGreenMail

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @BeforeAll
  public static void startGreenMail() {
      Properties props = new Properties();
      try (InputStream propStream = ExceptionMapperITCase.class.getResourceAsStream("/mail.properties")) {
          props.load(propStream);
      } catch (Exception e) {
          LOG.error("Could not load /mail.properties", e);
      }
  
      SMTP_HOST = props.getProperty("smtpHost");
      assertNotNull(SMTP_HOST);
      SMTP_PORT = Integer.parseInt(props.getProperty("smtpPort"));
      assertNotNull(SMTP_PORT);
  
      ServerSetup[] config = new ServerSetup[2];
      config[0] = new ServerSetup(SMTP_PORT, SMTP_HOST, ServerSetup.PROTOCOL_SMTP);
      config[1] = new ServerSetup(POP3_PORT, POP3_HOST, ServerSetup.PROTOCOL_POP3);
      greenMail = new GreenMail(config);
      greenMail.start();
  }
  ```

  开发者ID:apache，项目名称:syncope，代码行数:21，代码来源:[AbstractNotificationTaskITCase.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fapache%2Fsyncope)

- 示例7: setUp

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @Before
  public void setUp() throws MessagingException, UserException, InterruptedException {
      ServerSetup sS = new ServerSetup(4008, "localhost", ServerSetup.PROTOCOL_IMAPS);
      greenMail = new GreenMail(sS);
      greenMail.start();
      user = greenMail.setUser("[email protected]", "okkopa", "soooosecret");
      Properties props = System.getProperties();
      props.setProperty("mail.store.protocol", "imaps");
      Session session = Session.getDefaultInstance(props, null);
      MimeMessage message = new MimeMessage(session);
      message.setSubject("subject2576Hf");
      message.setText("viesti");
      user.deliver(message);
  
      Security.setProperty("ssl.SocketFactory.provider", DummySSLSocketFactory.class.getName());
  
      server = new IMAPserver("localhost", "okkopa", "soooosecret", 4008);
      server.login();
  
      IMAPfolder = new IMAPfolder(server, "inbox");
  
      assertTrue(greenMail.waitForIncomingEmail(5000, 1));
  
  }
  ```

  开发者ID:ohtuprojekti，项目名称:OKKoPa，代码行数:25，代码来源:[IMAPfolderTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fohtuprojekti%2FOKKoPa)

- 示例8: setUp

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @Before
  public void setUp() throws Exception {
      ServerSetup sS = new ServerSetup(4008, "localhost", ServerSetup.PROTOCOL_IMAPS);
      greenMail = new GreenMail(sS);
      greenMail.start();
      user = greenMail.setUser("[email protected]", "okkopa", "soooosecret");
      Properties props = System.getProperties();
      props.setProperty("mail.store.protocol", "imaps");
      Session session = Session.getDefaultInstance(props, null);
      MimeMessage message = new MimeMessage(session);
      message.setSubject("subject2576Hf");
      message.setText("viesti");
      user.deliver(message);
  
      Security.setProperty("ssl.SocketFactory.provider", DummySSLSocketFactory.class.getName());
  
      assertTrue(greenMail.waitForIncomingEmail(5000, 1));
  }
  ```

  开发者ID:ohtuprojekti，项目名称:OKKoPa，代码行数:19，代码来源:[IMAPserverTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fohtuprojekti%2FOKKoPa)

- 示例9: setUp

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @Before
  public void setUp() throws MessagingException, UserException, InterruptedException {
      Security.setProperty("ssl.SocketFactory.provider", DummySSLSocketFactory.class.getName());
  
      ServerSetup sS = new ServerSetup(4008, "localhost", ServerSetup.PROTOCOL_IMAPS);
      greenMail = new GreenMail(sS);
      greenMail.start();
      user = greenMail.setUser("[email protected]", "okkopa", "soooosecret");
      Properties props = System.getProperties();
      props.setProperty("mail.store.protocol", "imaps");
      Session session = Session.getDefaultInstance(props, null);
      MimeMessage message = new MimeMessage(session);
      message.setSubject("subject2576Hf");
      message.setText("viesti");
      user.deliver(message);
  
      assertTrue(greenMail.waitForIncomingEmail(5000, 1));
  
      server = new IMAPserver("localhost", "okkopa", "soooosecret", 4008);
      server.login();
  
      IMAPfolder = new IMAPfolder(server, "inbox");
  }
  ```

  开发者ID:ohtuprojekti，项目名称:OKKoPa，代码行数:24，代码来源:[IMAPmessageTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fohtuprojekti%2FOKKoPa)

- 示例10: startGreenMailServers

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  private GreenMail startGreenMailServers() {
      ServerSetup[] serverSetups = getServerSetups();
      GreenMail greenMail = new GreenMail(serverSetups);
      greenMail.setUser(PropertyAccessor.getTestMailBoxLogin(), PropertyAccessor.getTestMailBoxPass());
      greenMail.start();
      return greenMail;
  }
  ```

  开发者ID:tapack，项目名称:satisfy，代码行数:8，代码来源:[FakeEmailServerRunnerImpl.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Ftapack%2Fsatisfy)

- 示例11: testStatusCommand

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @Test
  public void testStatusCommand() {
      int smtpPort = SocketUtils.findAvailableTcpPort();
      ServerSetup setup = new ServerSetup(smtpPort, null, ServerSetup.PROTOCOL_SMTP);
      setup.setServerStartupTimeout(5000);
      GreenMail mailServer = new GreenMail(setup);
      mailServer.start();
      ((JavaMailSenderImpl) mailSender).setPort(smtpPort);
      sshCallShell((is, os) -> {
          write(os, "status");
          verifyResponse(is, "{\r\n  \"status\" : \"UP\"\r\n}");
          mailServer.stop();
      });
  }
  ```

  开发者ID:anand1st，项目名称:sshd-shell-spring-boot，代码行数:15，代码来源:[SshdShellAutoConfigurationTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fanand1st%2Fsshd-shell-spring-boot)

- 示例12: smtpServer

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  /**
   * Mock a smtp server.
   * @return GreenMail smtp server.
   * @throws IOException If something goes wrong.
   */
  public GreenMail smtpServer(String bind, int port) throws IOException {
      return new GreenMail(
          new ServerSetup(
              port, bind,
              ServerSetup.PROTOCOL_SMTP
          )
      );
  }
  ```

  开发者ID:opencharles，项目名称:charles-rest，代码行数:14，代码来源:[SendEmailTestCase.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fopencharles%2Fcharles-rest)

- 示例13: setupControllerTest

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @BeforeClass
  protected void setupControllerTest() {
      greenMail = new GreenMail(
              new ServerSetup[] {
                      new ServerSetup(30000, "127.0.0.1", ServerSetup.PROTOCOL_SMTP)
              }
      );
      greenMail.start();
  }
  ```

  开发者ID:passion1014，项目名称:metaworks_framework，代码行数:10，代码来源:[RegisterCustomerControllerTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fpassion1014%2Fmetaworks_framework)

- 示例14: setupEmailTest

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @BeforeClass
  protected void setupEmailTest() {
      greenMail = new GreenMail(
              new ServerSetup[] {
                      new ServerSetup(30000, "127.0.0.1", ServerSetup.PROTOCOL_SMTP)
              }
      );
      greenMail.start();
  }
  ```

  开发者ID:passion1014，项目名称:metaworks_framework，代码行数:10，代码来源:[EmailTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fpassion1014%2Fmetaworks_framework)

- 示例15: setUp

  ```java
  import com.icegreen.greenmail.util.GreenMail; //导入依赖的package包/类
  @BeforeClass
  public static void setUp() throws Exception {
      ServerSetup serverSetup = new ServerSetup(30993, "localhost", "imap");
      greenMail = new GreenMail(serverSetup);
      GreenMailUser user = greenMail.setUser("[email protected]", "[email protected]", "password");
      inbox = greenMail.getManagers().getImapHostManager().getInbox(user);
      greenMail.start();
  }
  ```

  开发者ID:theparanoidtimes，项目名称:tabellarium，代码行数:9，代码来源:[ImapMailboxFolderTaskExecutorTest.java](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Ftheparanoidtimes%2Ftabellarium)
