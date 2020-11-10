# Git 错误汇总

错误

```
error: index-pack died of signal 15
fatal: index-pack failed

	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandIn(CliGitAPIImpl.java:2450)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandWithCredentials(CliGitAPIImpl.java:2051)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.access$500(CliGitAPIImpl.java:84)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl$1.execute(CliGitAPIImpl.java:573)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl$2.execute(CliGitAPIImpl.java:802)
	at hudson.plugins.git.GitSCM.retrieveChanges(GitSCM.java:1219)
	at hudson.plugins.git.GitSCM.checkout(GitSCM.java:1297)
	at hudson.scm.SCM.checkout(SCM.java:505)
	at hudson.model.AbstractProject.checkout(AbstractProject.java:1206)
	at hudson.model.AbstractBuild$AbstractBuildExecution.defaultCheckout(AbstractBuild.java:574)
	at jenkins.scm.SCMCheckoutStrategy.checkout(SCMCheckoutStrategy.java:86)
	at hudson.model.AbstractBuild$AbstractBuildExecution.run(AbstractBuild.java:499)
	at hudson.model.Run.execute(Run.java:1894)
	at hudson.model.FreeStyleBuild.run(FreeStyleBuild.java:43)
	at hudson.model.ResourceController.execute(ResourceController.java:97)
	at hudson.model.Executor.run(Executor.java:428)
ERROR: Error cloning remote repo 'origin'
Finished: FAILURE
```

原因：内存不足

解决方案：

```
git gc
git pull
```

