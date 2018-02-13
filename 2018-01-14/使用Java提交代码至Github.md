**JGIT 实现java代码的git client操作**

**pom依赖**

> \<dependency\>
>
> \<groupId\>org.eclipse.jgit\</groupId\>
>
> \<artifactId\>org.eclipse.jgit\</artifactId\>
>
> \<version\>4.4.1.201607150455-r\</version\>
>
> \</dependency\>

**示例测试代码**

> package test;
>
> import static org.junit.Assert.fail;
>
> import java.io.File;
>
> import java.io.IOException;
>
> import org.eclipse.jgit.api.CreateBranchCommand.SetupUpstreamMode;
>
> import org.eclipse.jgit.api.Git;
>
> import org.eclipse.jgit.api.errors.GitAPIException;
>
> import org.eclipse.jgit.api.errors.JGitInternalException;
>
> import org.eclipse.jgit.internal.storage.file.FileRepository;
>
> import org.eclipse.jgit.lib.Repository;
>
> import org.junit.After;
>
> import org.junit.AfterClass;
>
> import org.junit.Before;
>
> import org.junit.BeforeClass;
>
> import org.junit.Test;
>
> public class TestJGit {
>
> private String localPath, remotePath;
>
> private Repository localRepo;
>
> private Git git;
>
> @BeforeClass
>
> public static void setUpBeforeClass() throws Exception {
>
> }
>
> @AfterClass
>
> public static void tearDownAfterClass() throws Exception {
>
> }
>
> @Before
>
> public void setUp() throws Exception {
>
> localPath = \"/Users/jack/gitcode/jgit\";
>
> remotePath = \"git@gitlab.xxxx.com:esigngroup/lhotse\_hsftest.git\";
>
> localRepo = new FileRepository(localPath + \"/.git\");
>
> git = new Git(localRepo);
>
> }
>
> @Test
>
> public void testCreate() throws IOException {
>
> Repository newRepo = new FileRepository(localPath + \"/.git\");
>
> newRepo.create();
>
> }
>
> @Test
>
> public void testClone() throws IOException, GitAPIException {
>
> Git.cloneRepository().setURI(remotePath).setBranch(\"finance\")
>
> .setDirectory(new File(localPath)).call();
>
> }
>
> @Test
>
> public void testPull() throws IOException, GitAPIException {
>
> git.pull().setRemoteBranchName(\"finance\").call();
>
> }
>
> @Test
>
> public void testAdd() throws IOException, GitAPIException {
>
> File myfile = new File(localPath + \"/myfile\");
>
> myfile.createNewFile();
>
> git.add().addFilepattern(\"myfile\").call();
>
> }
>
> @Test
>
> public void testCommit() throws IOException, GitAPIException,
>
> JGitInternalException {
>
> git.commit().setMessage(\"Added myfile\").call();
>
> }
>
> @Test
>
> public void testPush() throws IOException, JGitInternalException,
>
> GitAPIException {
>
> git.push().call();
>
> }
>
> @Test
>
> public void testTrackMaster() throws IOException, JGitInternalException,
>
> GitAPIException {
>
> git.branchCreate().setName(\"master\")
>
> .setUpstreamMode(SetupUpstreamMode.SET\_UPSTREAM)
>
> .setStartPoint(\"origin/master\").setForce(true).call();
>
> }
>
> @After
>
> public void tearDown() throws Exception {
>
> }
>
> @Test
>
> public void test() {
>
> fail(\"Not yet implemented\");
>
> }
>
> }

**完整代码**

> package com.steda.spider;
>
> import org.eclipse.jgit.api.Git;
>
> import org.eclipse.jgit.internal.storage.file.FileRepository;
>
> import org.eclipse.jgit.lib.Repository;
>
> import java.io.File;
>
> public class CodeDownloaderGIT implements CodeDownloader {
>
> private String localPath, remotePath, branch;
>
> private Repository localRepo;
>
> private Git git;
>
> // TODO git代码下载
>
> public String download(String appName, String codeUrl) {
>
> try {
>
> //代码拷贝目录
>
> localPath = System.getProperty(\"user.home\") + \"/\" + appName;
>
> localRepo = new FileRepository(localPath + \"/.git\");
>
> git = new Git(localRepo);
>
> String\[\] gitCodeUrl = codeUrl.split(\"::\");
>
> if (gitCodeUrl.length != 2) {
>
> return \"git代码地址请按照格式:
> git@gitlab.\[host\].com:esigngroup/lhotse\_hsftest.git::\[branch\]\";
>
> }
>
> remotePath = gitCodeUrl\[0\]; //
> git@gitlab.\[host\].com:esigngroup/lhotse\_hsftest.git::\[branch\]
>
> branch = gitCodeUrl\[1\];
>
> File localPathFile = new File(localPath);
>
> // 如果没有该代码目录,就执行git clone
>
> if (!localPathFile.exists()) {
>
> gitClone(remotePath, branch, localPath);
>
> } else { // 如果有代码,git pull
>
> gitPull(branch);
>
> }
>
> } catch (Exception e) {
>
> e.printStackTrace();
>
> }
>
> return localPath;
>
> }
>
> private void gitClone(String remotePath, String branch, String copyPath) {
>
> try {
>
> Git.cloneRepository().setURI(remotePath).setBranch(branch)
>
> .setDirectory(new File(copyPath)).call();
>
> } catch (Exception e) {
>
> e.printStackTrace();
>
> }
>
> }
>
> private void gitPull(String branch) {
>
> try {
>
> git.pull().setRemoteBranchName(branch).call();
>
> } catch (Exception e) {
>
> e.printStackTrace();
>
> }
>
> }
>
> }

**Porcelain**

The plumbing APIs are rather complete, but it can be cumbersome to
string them together to achieve common goals, like adding a file to the
index, or making a new commit. JGit provides a higher-level set of APIs
to help out with this, and the entry point to these APIs is the Git
class:

> Repository repo;
>
> // construct repo\...
>
> Git git = new Git(repo);

The Git class has a nice set of high-level builder-style methods that
can be used to construct some pretty complex behavior. Let's take a look
at an example -- doing something like git ls-remote:

> CredentialsProvider cp = new UsernamePasswordCredentialsProvider(\"username\", \"p4ssw0rd\");
>
> Collection\<Ref\> remoteRefs = git.lsRemote()
>
> .setCredentialsProvider(cp)
>
> .setRemote(\"origin\")
>
> .setTags(true)
>
> .setHeads(false)
>
> .call();
>
> for (Ref ref : remoteRefs) {
>
> System.out.println(ref.getName() + \" -\>
> \" + ref.getObjectId().name());
>
> }

This is a common pattern with the Git class; the methods return a
command object that lets you chain method calls to set parameters, which
are executed when you call .call(). In this case, we're asking the
origin remote for tags, but not heads. Also notice the use of
aCredentialsProvider object for authentication.\
Many other commands are available through the Git class, including but
not limited to add,blame, commit, clean, push, rebase, revert, and
reset.

**遇到的坑UnknownHostKey Exception in Accessing GitHub Securely**

加上一句:

> SshSessionFactory.setInstance(new JschConfigSessionFactory() {
>
> @Override
>
> protected void configure(OpenSshConfig.Host hc, Session session) {
>
> session.setConfig(\"StrictHostKeyChecking\", \"no\");
>
> }
>
> });
>
> package com.steda.spider;
>
> import com.jcraft.jsch.Session;
>
> import org.eclipse.jgit.api.Git;
>
> import org.eclipse.jgit.internal.storage.file.FileRepository;
>
> import org.eclipse.jgit.lib.Repository;
>
> import org.eclipse.jgit.transport.\*;
>
> import java.io.File;
>
> public class CodeDownloaderGIT implements CodeDownloader {
>
> private String localPath, remotePath, branch;
>
> private Repository localRepo;
>
> private Git git;
>
> // TODO git代码下载
>
> public String download(String appName, String codeUrl) {
>
> try {
>
> //代码拷贝目录
>
> localPath = System.getProperty(\"user.home\") + \"/\" + appName;
>
> localRepo = new FileRepository(localPath + \"/.git\");
>
> git = new Git(localRepo);
>
> SshSessionFactory.setInstance(new JschConfigSessionFactory() {
>
> @Override
>
> protected void configure(OpenSshConfig.Host hc, Session session) {
>
> session.setConfig(\"StrictHostKeyChecking\", \"no\");
>
> }
>
> });
>
> String\[\] gitCodeUrl = codeUrl.split(\"::\");
>
> if (gitCodeUrl.length != 2) {
>
> return \"git代码地址请按照格式:
> git@gitlab.\[host\].com:esigngroup/lhotse\_hsftest.git::\[branch\]\";
>
> }
>
> remotePath = gitCodeUrl\[0\]; //
> git@gitlab.\[host\].com:esigngroup/lhotse\_hsftest.git::\[branch\]
>
> branch = gitCodeUrl\[1\];
>
> File localPathFile = new File(localPath);
>
> // 如果没有该代码目录,就执行git clone
>
> if (!localPathFile.exists()) {
>
> gitClone(remotePath, branch, localPath);
>
> } else { // 如果有代码,git pull
>
> gitPull(branch);
>
> }
>
> } catch (Exception e) {
>
> e.printStackTrace();
>
> }
>
> return localPath;
>
> }
>
> private void gitClone(String remotePath, String branch, String copyPath) {
>
> try {
>
> //.setCredentialsProvider(cp)
>
> Git.cloneRepository().setURI(remotePath).setBranch(branch)
>
> .setDirectory(new File(copyPath)).call();
>
> } catch (Exception e) {
>
> e.printStackTrace();
>
> }
>
> }
>
> private void gitPull(String branch) {
>
> try {
>
> git.pull().setRemoteBranchName(branch).call();
>
> } catch (Exception e) {
>
> e.printStackTrace();
>
> }
>
> }
>
> }

![Alt text](./1515923245708.png)
