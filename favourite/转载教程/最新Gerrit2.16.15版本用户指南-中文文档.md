[出处](这是为Gerrit最终用户准备的Gerrit指南。 它说明了标准的Gerrit工作流程以及指导用户可以根据个人喜好来设置并使用Gerrit。

为了更好地理解本指南，读者最好了解Git，并熟悉基本的git命令和工作流程。

什么是Gerrit
Gerrit是一个Git服务器，为托管的Git存储库提供访问控制，并提供Web前端进行代码审查。 代码审查是Gerrit的核心功能，但仍然是可选的，团队可以决定直接使用但不进行代码审查。

工具
Gerrit使用git协议。这意味着要使用Gerrit，您不需要安装任何Gerrit客户端，只要拥有常规的git客户端（例如git命令行或Eclipse中的EGit）就足够了。

仍然有一些Gerrit客户端工具，可以选择使用这些工具：

Mylyn Gerrit Connector：Mylyn集成的Gerrit
Gerrit IntelliJ插件：IntelliJ平台集成的Gerrit
mGerrit：Gerrit的Android客户端
Gertty：基于控制台界面的Gerrit
克隆Gerrit项目
克隆Gerrit项目的方法与使用git clone命令克隆任何其他git存储库的方法相同。

克隆Gerrit项目：

$ git clone ssh://gerrithost:29418/RecipeBook.git RecipeBook
  Cloning into RecipeBook...
BashCopy
可以在Gerrit Web UI中的Projects > List > <project-name> > General下找到克隆项目的URL。

译者注：上面的UI为旧的GWT UI路径，2.16.15默认为PolyGerrit UI，3.0以后只有新的PolyGerrit UI界面。新UI界面路径为BROWSE > Repositories > <project-name>，打开后最上方即为克隆路径。

对于git操作，Gerrit支持SSH和HTTP/HTTPS协议。

注意

要使用SSH，您可能需要在Settings中配置SSH公钥。

代码审查工作流
使用Gerrit进行代码审查意味着先审查每个提交，然后再将其合入到代码库中。

代码修改者对代码的每一次commit在Gerrit中都作为一个change。 在Gerrit中，每个change都存储在暂存区域中，我们可以对暂存区中的代码进行检查和审核。只有当这部分代码审核通过并且点击submitt按钮后，它才会被合并到代码库中。

如果有关于change的反馈（其他reviewer对你提交的代码提出了修改意见），那么作者可以通过amend commit来修改代码并提交新的补丁集。 这样，每次提交的change就可以迭代地得到改进，并且仅在准备就绪时才将其应用于代码库中。

上传一个change
通过对Gerrit推送一次提交就完成了一个change的上传。（这是一个很重要的概念，后面的大部分论述都以change为操作对象） 必须将提交推送到使用目标分支名命名的引用分支：refs/for/<target-branch>。 神奇的refs/for/前缀使Gerrit可以区分这次提交是需要代码审核的提交，还是不用审核直接合入仓库的提交。

对于目标分支，只需指定短名称即可，例如master，但您也可以指定完全限定的分支名称，就像refs/heads/master这样。

需要审核的推送：

$ git commit
$ git push origin HEAD:refs/for/master

// this is the same as:
$ git commit
$ git push origin HEAD:refs/for/refs/heads/master
BashCopy
绕过审核的推送：

$ git commit
$ git push origin HEAD:master

// this is the same as:
$ git commit
$ git push origin HEAD:refs/heads/master
BashCopy
注意

如果推送到Gerrit失败，请查阅Gerrit文档以了解错误消息。

推送提交进行审查时，Gerrit将其存储在暂存区域中，该暂存区域是特殊refs/changes/名称空间中的一个分支。 更改参考的格式为refs/changes/XX/YYYY/ZZ，其中YYYY是数字更改号，ZZ是补丁集编号，而XX是数字更改号的后两位，例如refs/changes/20/884120/1。使用Gerrit并不需要了解此引用的格式。

获取更新

$ git fetch https://gerrithost/myProject refs/changes/74/67374/2 && git checkout FETCH_HEAD
BashCopy
注意：
可以从你的更新提交界面中直接下载命令复制fetch命令。

refs/for/前缀用于将Gerrit“推送到审核”概念映射到git协议。对于git客户端，似乎每次推送都转到同一个分支，例如refs/for/master，但实际上对于推送到此ref的每个提交，Gerrit都会在refs/changes/名称空间下创建一个新分支。此外，Gerrit还创建了一个开放的change。

change包括一个Change-Id，元数据（所有者，项目，目标分支等），一个或多个补丁集，注释和投票。一个补丁集是一次git commit。change中的每个补丁集代表该change的新版本，并替换了先前的补丁集。仅最新的补丁集是相关的。这意味着change的所有失败迭代将永远不会应用于目标分支，而只会集成最后批准的补丁集。

对于Gerrit来说，Change-Id很重要，因为它知道为代码检查而推动的提交是否应该创建新的change，或者是否应该为现有change创建新的补丁集。

Change-Id是SHA-1，其前缀为大写字母I。在提交消息（最后一段）中将其指定为页脚：

Improve foo widget by attaching a bar.

We want a bar, because it improves the foo by providing more
wizbangery to the dowhatimeanery.

Bug: #42
Change-Id: Ic8aaa0728a43936cd4c6e1ed590e01ba8f0fbf5b
Signed-off-by: A. U. Thor <author@example.com>
BashCopy
如果将提交消息中具有Change-Id的提交推送进行审查，则Gerrit会检查该项目和目标分支是否已存在具有此Change-Id的change，如果是，则Gerrit会为此change创建一个新的补丁集 。 如果不是，则使用给定的Change-Id创建新的change。

如果将没有Change-Id的提交推送进行审查，则Gerrit会创建一个新change并为其生成一个Change-Id。 由于在这种情况下，Change-Id不包含在提交消息中，因此在上传新补丁集时必须手动插入它。在推入第一个补丁集时，大多数项目已经需要一个Change-Id。这样可以减少意外创建新change而不是上传新补丁集的风险。 如果没有Change-Id，则任何推送都会失败，因为提交消息页脚中缺少Change-Id。

修改并重新确定提交将保留Change-Id，以便在将其提交进行审阅时，新提交将自动成为现有更改的新补丁集。

推送一次新的补丁集：

$ git commit --amend
$ git push origin HEAD:refs/for/master
BashCopy
Change-Ids对于项目的分支是唯一的。例如，在不同分支中解决相同问题的提交应具有相同的Change-Id，如果将提交pick到另一个分支中，则提交会自动发生。这样，您可以使用Gerrit Web UI在所有分支中按Change-Id搜索，找到你的修复代码。

可以通过按照Change-Id文档中的说明安装commit-msg钩子来自动创建Change-Id。

您可以将其复制到git模板目录中，而不是为每个git存储库手动安装commit-msg钩子。然后将其自动复制到每个新克隆的存储库中。

审查change
上传更改以供审核后，审阅者可以通过Gerrit Web UI进行检查。审阅者可以查看代码增量，并直接在代码块或代码行中进行注释。他们还可以发布摘要评论并在审核标签上投票。审阅用户UI的文档说明了进行代码审阅的步骤与操作。

有几个选项可控制应如何呈现补丁差异。 用户可以在diff首选项中配置其首选项。

上传一个新的补丁集
如果有来自代码审查的反馈，并且应改进更改，则应上传带有重新编写的代码的新补丁集。

这可以通过修改最后一个补丁集的提交来完成。如果需要，可以通过使用change页面中的fetch命令从Gerrit中获取此提交。

重要的是，提交消息中包含应作为页脚更新的更改的Change-Id（最后一段）。 通常，提交消息已经包含正确的Change-Id，并且在修改提交时保留了Change-Id。

推送一次补丁集：

// fetch and checkout the change
// (checkout command copied from change screen)
$ git fetch https://gerrithost/myProject refs/changes/74/67374/2 && git checkout FETCH_HEAD

// rework the change
$ git add <path-of-reworked-file>
...

// amend commit
$ git commit --amend

// push patch set
$ git push origin HEAD:refs/for/master
BashCopy
注意：永远不要修改已经属于中央分支的提交。

推送新补丁集将触发电子邮件通知审阅者。

并行开发多个功能
代码审查需要时间，提交更新的作者可以使用它来实现其他功能。

每个功能都应在其自己的本地功能分支中实现，该分支基于目标分支的当前HEAD。这样，就不必依赖其他尚未审核通过的change，并且可以独立查看和应用新功能。如果需要，也可以将新功能基于其他尚未审核通过的change。这将在Gerrit的change之间创建依赖关系，并且只有同时应用了所有change，才能应用每个change。change之间的依赖关系可以从change页面上的Related Changes选项卡中看到。

查看项目
要了解新的更改，您可以查看您感兴趣的项目。对于查看的项目，Gerrit在上传或修改change后会向您发送电子邮件进行通知。您可以决定要通知哪些事件，也可以使用更改搜索表达式来过滤通知。例如，'branch:master file:^.*\.txt$'将仅向master分支中涉及到”txt”文件的更改发送电子邮件通知。

通常，项目团队的成员查看他们自己的项目，然后选择他们感兴趣的更改进行审查。

项目所有者还可以在项目级别配置通知。

增加新的代码审核者
在change页面中，可以将审阅者明确添加到change中。然后，将通过电子邮件通知添加的审阅者进行审阅。

主要是，此功能用于请求对已知是修改后的代码方面的专家或所实现功能的涉众的特定人员进行审阅。通常，不需要显式地为每个更改添加审阅者，而是您需要依靠项目团队来监视他们的项目并根据重要性，兴趣，时间等来处理即将到来的更改。

还有一些插件可以自动添加评论者（例如，通过配置或基于git blame注释）。如果需要此功能，应与项目所有者和Gerrit管理员进行讨论。

仪表板
Gerrit支持多种查询运算符，可以根据不同的条件（例如，根据状态，更改所有者，投票等）来查询更改。

显示更改查询结果的页面的URL中包含了这次查询的条件。这意味着您可以在浏览器中为该URL添加书签以保存更改查询。这样，以后可以很容易地重新执行它。

几个变更查询也可以合并到仪表板中。仪表板是Gerrit中的一个页面，用于显示不同部分中的多个更改查询的结果，每个部分具有描述性标题。

My > Changes提供了默认的仪表板(新UI默认主页就是仪表板)。它可以显示已经完成的审核、需要进行的审核和最近关闭的change。

用户还可以定义自定义仪表板。可以在浏览器中为仪表板添加书签，以便以后可以重新执行。

还可以自定义My菜单，并向其中添加自定义查询或仪表板的菜单项。

仪表板对于定义自己的change视图非常有用，例如您可以为不同的项目集使用不同的仪表板。

注意：您可以使用limit和age查询运算符来限制仪表板查询结果。单击章节标题将执行不带limit和age运算符的更改查询，以便您可以检查整个结果集。

提交change
提交change意味着将当前补丁集的代码修改应用于目标分支。提交需要Submmit权限，并且可以通过单击submmit按钮在更改屏幕上完成。

为了能够提交change，必须首先通过在审阅标签上投票批准change。默认情况下，仅当change在每个审阅标签上具有最高值的投票而没有最低值（否决票）的投票时，才能提交更改。项目可以配置自定义标签和自定义提交规则，以控制更改何时可提交。

提交change时，如何将代码修改应用于目标分支，由可在项目级别配置的提交类型控制。

提交change可能会因冲突而失败。在这种情况下，您需要在本地重新设置change，解决冲突并将具有冲突解决方案的提交上传为新补丁集。

如果由于路径冲突而无法合并change，则会在更改屏幕上以红色粗体Cannot Merge标签突出显示该change。

变基change
在检查change时，目标分支的HEAD可能会更新。在这种情况下，change可以重新基于目标分支的新HEAD。如果没有冲突，则可以直接在change页面上进行变基，否则必须在本地完成。

在本地进行变基：

// update the remote tracking branches
$ git fetch

// fetch and checkout the change
// (checkout command copied from change screen)
$ git fetch https://gerrithost/myProject refs/changes/74/67374/2 && git checkout FETCH_HEAD

// do the rebase
$ git rebase origin/master

// resolve conflicts if needed and stage the conflict resolution
...
$ git add <path-of-file-with-conflicts-resolved>

// continue the rebase
$ git rebase --continue

// push the commit with the conflict resolution as new patch set
$ git push origin HEAD:refs/for/master
BashCopy
仅当存在Gerrit无法解决的冲突时，才需要进行手动变基。如果需要手动解决冲突，则还取决于为项目配置的提交类型。

通常，change不应无故变基，因为它会增加补丁集的数量并在发出通知时产生噪音。但是，如果对change进行了长时间审核，则可能需要不时重新设置其基准，以便审核者可以查看目标分支当前HEAD的增量。它还表明对这个change仍然有兴趣。

注意：切勿对已经是中央分支一部分的提交进行变基。

放弃/恢复change
有时，在代码检查期间发现change很不好，应该放弃。在这种情况下，可以放弃change，以使它不再出现在未完成的change列表中。

如果以后需要再次使用，可以恢复被放弃的change。

使用主题
change可以按主题分组。 这很有用，因为它使您可以使用主题搜索运算符轻松找到相关的change。此外，在change页面上还会显示具有相同主题的change，以便您可以轻松地在它们之间导航。

通常实现同一个功能或用户需求的change按主题分组。

可以在change页面中为change分配主题。

也可以通过在引用名称后附加%topic=…或通过使用命令行标志--push-option（别名为-o，后面加上topic=…）来设置推送主题。

Gerrit可以配置为一次单击即可提交主题中的所有change，即使主题跨越多个项目也是如此。

通过push来设置主题：

$ git push origin HEAD:refs/for/master%topic=multi-master

// this is the same as:
$ git push origin HEAD:refs/heads/master -o topic=multi-master
BashCopy
使用hashtags
hashtags是与change相关联的自由格式字符串，类似社交媒体平台上的用户标签。 在Gerrit中，您可以使用UI的专用区域将hashtags与change改明确关联。 它们不会从提交消息或注释中解析。

与主题类似，hashtags可用于将相关change分组在一起，并使用hashtag:运算符进行搜索。 与主题不同，change可以具有多个hashtags，并且仅用于信息分组。 具有相同标签的更改不必一起提交。

仅在NoteDb下运行时，hashtags功能才可用。

通过push来设置hashtags：

$ git push origin HEAD:refs/for/master%t=stable-bugfix

// this is the same as:
$ git push origin HEAD:refs/heads/master -o t=stable-bugfix
BashCopy
Work-in-Progresschange
任何人都可以看到进行中的（WIP）change，但不会通知审阅者或要求审阅者采取任何措施。

具体来说，当您将更改标记为”Work-in-Progress”时：
– 大多数操作都不会通知审阅者，例如添加或删除，发布评论等。有关更多信息，请参见REST API文档表。
– change不会显示在审阅者的仪表板上。

在以下情况下，WIP更改非常有用：
– 您仅实现了change的一部分，但是想要在请求审阅者反馈之前将change推送到服务器以运行测试或执行其他操作。
– 在审阅过程中，您意识到您需要对change进行新的处理，并且您想要在完成更新之前停止将更改通知给审阅者。

要将change的状态设置为”Work-in-Progress”，可以使用命令行或用户界面。要使用命令行，请将%wip附加到您的推送请求中。

$ git push origin HEAD:refs/for/master%wip
BashCopy
或者，从change页面中单击WIP。

要change标记为可供审核，请将%ready附加到您的推送请求中。

$ git push origin HEAD:refs/for/master%ready
BashCopy
或者，从change页面单击Ready。

只有change所有者，项目所有者和站点管理员才能将change标记为work-in-progress和Ready。

在新的PolyGerrit用户界面中，您可以通过从More菜单中选择WIP来将更改标记为WIP。 change页面以黄色标题更新，表明change页面处于work-in-progress状态。要将change标记为可以审阅，请单击Start Review。

私有change
私有change是只有change所有者、审阅者和拥有全局查看私更改功能的用户才能看到的change。私有change在许多情况下很有用：

您希望一组协作者在正式审核开始之前就审核change。通过创建一个私有change并仅添加选定的少数几个审阅者，您可以控制哪些人可以看到该change并获得第一意见，然后再向所有审阅者开放。
您想要在正式审查开始之前检查change的显示。通过将change标记为私有而没有审阅者，没有人可以过早地评论您的change。
您想使用Gerrit在不同设备之间同步数据。通过创建不带审阅者的私有change，您可以从一台设备推送并在另一台设备上获取。
您想要对敏感的change进行代码审查。通过在私有change中查看安全修复程序，外部人员无法在推出安全修复程序之前查看到该修复。即使在合并change后，该审查也可以保持私有。
要创建私有change，请使用private选项进行推送。

推送私有change：

$ git commit
$ git push origin HEAD:refs/for/master%private
BashCopy
除非您指定remove-private选项，否则change将在随后的推送中保持私有状态。再者，Web UI提供按钮以再次将change标记为私有和非私有。

当使用由另一个用户提交的commit来推送私有change时，这个用户不会自动被添加为审阅者，而是必须明确添加。

对于必须验证私有change的CI系统，可以授予特殊权限（查看私有change）。在这种情况下，应注意防止CI系统公开秘密细节。

忽略或标记change为“已审阅”
change可以忽略，这意味着它们将不会出现在“即将进行的审阅”（Incoming Reviews）仪表板中，并且任何相关的电子邮件通知都将被取消。当您作为审阅者不想积极参与审阅，但又不想让自己退出该change的审阅时，此功能很有用。

或者，可以将其标记为“已审核”（Reviewed），而不是完全忽略该change。将change标记为“已审阅”意味着在上传新补丁集之前，该change不会在仪表板上突出显示。

内联编辑
可以直接在Web UI中内联编辑change。这对于立即进行小的更正并将其发布为新补丁集很有用。

也可以内联创建新change。

项目管理
每个项目都有一个项目所有者来管理该项目。

项目管理包括项目访问权限的配置，但是项目所有者拥有更多的可能性来定制项目的工作流，这在项目所有者指南中进行了描述。

不使用代码审查
使用Gerrit进行代码审查是可选的，您可以不使用代码审查就将Gerrit用作纯Git服务器。
绕过审核的推送：

$ git commit
$ git push origin HEAD:master

// this is the same as:
$ git commit
$ git push origin HEAD:refs/heads/master
BashCopy
在项目访问权限中启用绕过代码检查。项目所有者必须通过在目标分支（refs/heads/<branch-name>）上分配Push权限来允许它。

如果您选择不使用代码审查，请记住这一点。提交代码时，只要目标分支的HEAD已移动，则始终需要手动进行merge/rebase。您认为避免审查工作流程的额外复杂性会让代码提交与合入变得更容易，实际上可能并不是这样。

项目所有者可以设置在推送时自动合并，这样直接推送到存储库时可以在服务器端自动merge/rebase。

用户引用
用户配置数据（例如首选项preferences）存储在All-Users项目的每个用户引用下。用户的引用是基于用户的帐户ID（整数）。用户引用由引用名称中的后两位数字（nn）分片，从而导致引用格式为refs/users/nn/accountid。

首选项
在Gerrit Web UI里有多个选项可以控制最终的呈现效果。用户可以在Settings > Preferences下配置其首选项。用户的首选项存储All-Users项目的每个用户引用下的preferences.config文件中，该文件样式类似git config。

可以配置以下首选项：
– Display In Review Category:
此设置控制如何显示change列表和仪表板上的审阅标签的值。
– None:
对于每个审阅标签，仅显示投票值。批准显示为绿色复选标记图标，否决显示为红色X图标。
– Show Name:
对于每个审阅标签，将显示投票值以及投票用户的全名。
– Show Email:
对于每个审阅标签，将显示投票值以及投票用户的电子邮件地址。
– Show Username:
对于每个审阅标签，将显示投票值以及投票用户的用户名。
– Show Abbreviated Name:
对于每个审阅标签，将显示投票值以及投票用户全名的缩写。
– Maximum Page Size:
一页上显示的最大条目数，例如在change，项目，分支或组中进行分页时使用。
– Date/Time Format:
用于呈现日期和时间戳的格式。
– Email Notifications:
此设置控制电子邮件通知。
– Enabled:
电子邮件通知已启用。
– Every comment:
电子邮件通知已启用，并且您将在自己撰写的评论上以抄送的形式收到电子邮件通知。
– Disabled:
电子邮件通知已停用。
– Email Format:
此设置控制Gerrit发送的电子邮件格式。请注意，如果管理员已禁用Gerrit实例的HTML电子邮件，则此设置无效。
– Plaintext Only:
电子邮件通知仅包含纯文本内容。
– HTML and Plaintext:
电子邮件通知包含HTML和纯文本内容。
– Default Base For Merges:
当打开change页面以进行合并提交时，此设置控制应在Diff Against下拉列表中预先选择哪个base。
– Auto Merge:
当打开change页面以进行合并提交时，在Diff Against下拉列表中预先选择Auto Merge。
– First Parent:
当打开change页面以进行合并提交时，在Diff Against下拉列表中预先选择First Parent。
– Diff View:
单击change页面的文件路径时，应显示Side-by-Side还是Unified差异视图。
– Show Site Header / Footer:
是否应显示网站的页眉和页脚。
– Show Relative Dates In Changes Table:
更改列表和仪表板中的时间戳是否应显示为相对时间戳，例如“12天前”，而不是绝对时间戳，例如“4月15日”。
– Show Change Sizes As Colored Bars:
更改大小是否应显示为彩色条。如果禁用，则添加和删除的行数将显示为文本，例如 ‘+297，-63’。
– Show Change Number In Changes Table:
是否应在change列表和仪表板中显示带有数字变更ID的ID列。
– Mute Common Path Prefixes In File List:
change页面上文件列表中的公用路径前缀是否应显示为灰色。
– Insert Signed-off-by Footer For Inline Edit Changes:
是否应将Signed-off-by页脚自动插入到从web UI创建的更改中（例如，通过project页面上的Create Change和Edit Config按钮以及change页面上的Follow-Up按钮）。
– Publish comments on push:
推送更新到还处于open状态的change时，默认情况下是否发布任何未完成的草稿注释。这个首选项只是设置默认值。仍然可以使用push选项覆盖行为。
– Use Flash Clipboard Widget:
是否应使用Flash剪贴板小部件。如果启用了Flash插件并且Flash插件可用，那么Gerrit会在需要频繁复制的ID和命令（例如，更改ID，提交ID和下载命令）旁边提供一个“复制到剪贴板”图标。请注意，仅当Flash插件可用且JavaScript Clipboard API不可用时，才显示此选项。
– Set new changes work-in-progress:
默认情况下，是否将新更改作为work-in-progress上传。这个首选项只是设置默认值。仍然可以使用push选项覆盖行为。

另外，可以自定义My菜单的菜单项。这可以用于快速导航到常用页面，例如配置过的仪表板。
===
)