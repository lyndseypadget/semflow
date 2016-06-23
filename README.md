<p><img alt="" height="291" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/tree.jpg" width="800"></p>

<h2>Semflow: The SemVer Workflow</h2>
<h4>A Git Workflow for Version Management: RESTful APIs and beyond</h4>

<p>When I mention versioning, a lot of people assume I'm talking about how someone specifies the version of a RESTful API that he or she wants to consume. Namely, <a href="https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/">URI vs content negotiation</a>. That's not what this article is about, but it is requisite to remember that if you are asking your consumers specify which version they want, you are presumably offering (and to some degree maintaining) multiple versions at a time. If you only offered one version at a time, consumers wouldn't have to specify which one they want. That would be silly, because there's only one choice.</p>

<p>Maintaining many released versions of an API at a time is what this article is about. Specifically, how to do this with git from a release management point of view. That means seeing both the trees <em>and the forest </em>among your git branches. (You see what I did there with the branch pun?)</p>

<p>For the purpose of this article, we assume that the git repository only contains the RESTful API <em>specification</em> (i.e. RAML, Swagger or JSON API files) As such, I'm going focus only on things worthy of major or minor releases according to <a href="http://semver.org/">Semantic Versioning</a> (aka SemVer). Bug fixes which increment the patch number are related to the <em>incorrect implementation</em> of an unchanged specification, and therefore out of scope here.</p>

<h2>Git Branching Strategy Options</h2>

<p>When I mention git branching strategies, a lot of people assume I'm talking about <a href="https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow">the Gitflow workflow</a>. Some propose Gitflow as a solution for managing releases, or they describe <a href="https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow">the Feature Branch workflow</a> and call it Gitflow. Occasionally I'll even get a bit of mansplaining about what a branch is. So let's clear the air.</p>

<p><img alt="" height="336" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/mansplaining.jpg" width="336"></p>

<p>In a nutshell, the Feature Branch workflow dictates that feature-related work should happen on a branch dedicated for that purpose. There is often a common nomenclature for naming these branches. This almost seems like common sense nowadays, because the alternative is that everyone works in the <strong>master</strong> branch all the time. An API specification is no exception to this principle: its changes should be feature-driven and reviewed/tested before being merged. Feature branches are good.</p>

<p>The Gitflow workflow zooms out a bit, and defines a strict branching model for releasing projects. Vincent Driessen introduces this model in <a href="http://nvie.com/posts/a-successful-git-branching-model/">his blog post</a>. This strategy relies on two branches, <strong>master</strong> and <strong>develop</strong>, which have an infinite lifetime. Release branches exist temporarily and are deleted after merging into <strong>master</strong>. This is where you realize that Gitflow is perfect for many applications, as long as they only intend to release and maintain one version of their project at a time. <strong>master</strong> represents the current release, and <strong>develop</strong> represents the forthcoming one. You can hotfix the current release, and tags represent older releases, but at that point the code is all on the same <strong>master</strong> branch.</p>

<p>Given this, can you use Gitflow to track, deploy and troubleshoot multiple releases at a time? Yes. Can you use Gitflow to fix or enhance many releases at a time? <a href="http://stackoverflow.com/questions/16386323/following-git-flow-how-should-you-handle-a-hotfix-of-an-earlier-release">No.</a> We need a different model.</p>

<h2>Previous Versions: To Enhance or Not to Enhance</h2>

<p>When I mention enhancing older releases, a lot of people look at me like I'm crazy. “Why would you ever do that?” they ask, as though I'm the world's biggest sucker. Here's where you need to understand why that is necessary (common, even) in the world of RESTful APIs.</p>

<p>The following chart shows how an endpoint can change over time. Let's say you maintain a RESTful API for managing cars at a dealership. All of these are POST calls that accept a car model. I've defined this model in <a href="https://github.com/raml-org/raml-spec">RAML</a>, but I've only included the properties here for brevity.</p>

<p><img alt="" height="399" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/v_table.png" width="795"></p>

<p>If you're interested, here's how this history played out:</p>

<ul>
	<li>In v2, it became required to specify a <a href="https://en.wikipedia.org/wiki/Vehicle_identification_number">vin</a> when creating a car object. Without one, the request would fail. This is a breaking change, so the major version is bumped.</li>
	<li>In v2.1, a nice-to-have feature was implemented, which introduced the option to indicate whether or not the car has a CD player. If you omit this field, the POST call will still succeed. This is a passive change, so the minor version is bumped. v2 is no longer supported, because consumers using v2 can now use v2.1 without changing their code.</li>
	<li>In v3, the mileage field was introduced. Like vin, this is a breaking change. But there's another subtle change here: the optional hasCDPlayer field is no longer supported/accepted (because the only person who still uses CDs is my grandma). Whether or not <em>that</em> is a breaking or passive change depends on how the system behaves when it is given. You can throw an error - making it break - or you can silently ignore the now-unrecognized field. In this case, I recommend the latter because the field is optional, and you may want to release v2.2 without it.</li>
	<li>In v3.1, yet another optional field was added; bluetooth is all the rage now. Again, v3 is no longer supported because consumers who used it can now seamlessly use v3.1 instead. That's what makes v3.1 passive, by definition.</li>
</ul>

<p>Let's say that the tablet version of the dealership's car-searching app uses v2.1 and the desktop version uses v3.1. The salesmen at the dealership prefer using tablets, because they can quickly manage car inventory without leaving the customer on the floor. Now, the dealership would like the ability to add/search cars by interior color on the tablet app. This is a request to enhance v2.1 by adding a new optional field called interiorColor, thus making v2.2.</p>

<p><img alt="" height="354" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/v_22.png" width="165"></p>

<p>Should this be allowed? Some might say no – only introduce enhancements in the latest release and tell the client to upgrade. This seems unaccomodating. This client here is a car dealership, and they don't have the time or money for an entire v3 upgrade. They just want to add one little field to see if they can sell more cars.</p>

<p>In a perfect world, everyone would be on the latest version. But we don't live in a perfect world, which is why we support multiple versions in the first place. Perhaps such enhancements are experimental business strategies that can revolutionize or devastate the client's sales. I believe it's better to figure that out on a passive older release, so <em>you can decide </em>whether or not to include it in the next version.</p>

<h2>The SemVer Workflow</h2>

<p>The basic premise of the SemVer workflow is to keep each major release in its own branch.</p>

<p><strong>master</strong> is not special. &nbsp;When you create a new git repository, <strong>master</strong> is the just the recommended name for the first trunk-like branch you will create. &nbsp;<strong>master</strong> is also the default branch. &nbsp;However, until you make the first commit, the first branch doesn’t even exist.</p>

<p>Recall that <strong>master</strong> represents the last release and <strong>develop</strong> represents the forthcoming one. &nbsp;If you were really set on keeping those branch names, you could still use many aspects of Gitflow, but you would keep release branches indefinitely. &nbsp;When you release a new major version, it is tagged and merged into <strong>master</strong> and <strong>develop</strong>. &nbsp;When you release a new minor version, it is branched from the appropriate release branch.</p>

<p><img alt="" height="824" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/gitflow_hack.png" width="632"></p>

<p>This works, but personally, I think it’s confusing. &nbsp;If we’re going to maintain branches for each major release, I would rather each branch name be explicit. &nbsp;I don’t like having to wonder what major version these branches contain.</p>

<p>In a SemVer workflow, <strong>master</strong> and <strong>develop</strong> do not exist. &nbsp;The name of the “trunk” branch is <strong>v0</strong>. &nbsp;Minor releases get a tag (i.e. 1.1.0), while major versions get a tag (i.e. 2.0.0) and a new branch (i.e. v2). &nbsp;Code on each branch’s HEAD (the leaves in the tree) can still be hotfixed if necessary, and enhanced with code from feature branches.</p>

<p>This approach becomes especially useful when you have a feature off of a v(N) branch that you would like to merge into both v(N) and v(N+1). &nbsp;When the feature is complete, you can squash that branch’s commits and cherry-pick them over to other branches as needed, resolving conflicts if necessary. &nbsp;This might seem unlikely, but occasionally you begin a feature expecting both its need and impact to be isolated, but discover later that it could/should be applied more broadly.</p>

<p>For this reason, it’s recommended that you keep a set of code or specification changes separate from the commit that actually bumps the version number or applies a tag. &nbsp;In my case, I track the version number in the package.json file, and I don’t want that change to come along with the RAML modifications I want to apply in a different branch. &nbsp;Failing to keep these separate will result in a post cherry-pick conflict resolution like this:</p>

<p><img alt="" height="279" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/package_json_conflict.png" width="818"></p>

<p>Let’s say I want to bring the vehicle’s interiorColor field into version 3 of the API. &nbsp;In this example, I’m on the <strong>v3</strong> branch and cherry-picking a commit from the <strong>v2</strong> branch. &nbsp;&nbsp;If I wasn’t paying close attention, I might choose the wrong version, and things would get very ugly.<br>
	&nbsp;</p>

<h2>The SemVer Workflow in Action – An Example</h2>

<p>First, we'll create a branch that represents our first “beta” release – <strong>v0</strong>. This branch contains what will soon become v1 of the API. (If you've already checked in code to <strong>master</strong>, you can rename it to <strong>v0</strong> and delete the master branch!) Remember to set the default branch to <strong>v0</strong> in your [Stash] repository settings:</p>

<p><img alt="" height="345" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/set_default_branch.png" width="1254"></p>

<p>When you're ready to release version 1 of your API, you create a new branch. You can use <a href="https://docs.npmjs.com/cli/version">npm version</a> to tell the project how to increment itself according to SemVer rules, which will create an isolated commit with a tag. (Note that I should have pushed <em>with tags </em>here: git push origin v1 --tags).</p>

<p><img alt="" height="359" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/make_v1.png" width="638"></p>

<p>In this case, the RAML itself can have a version number, but I removed it and will rely on the package.json instead. &nbsp;(Scripts can help sync the two - the goal is to have a single source of truth for the version number.)</p>

<p>If I keep adding to the API spec like the example chart above indicates, my tree looks something like this:</p>

<p><img alt="" height="415" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/linear_tree.png" width="765"></p>

<p>This represents typical development where no minor updates are made to older releases. &nbsp;Notice how it looks linear, even though we’ve made branches for each major version. &nbsp;Then, when we update the <strong>v2</strong> branch with an enhancement (v2.2) the tree looks like this:</p>

<p><img alt="" height="407" src="https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/split_tree_after_2.2.png" width="781"></p>

<h2>Summary</h2>

<p>Git is a powerful and flexible tool. &nbsp;Standard workflows are fundamental in helping teams keep code organized, but they should never box you in. &nbsp;In general, the Feature Branch and Gitflow workflows are fantastic, but you can adapt them to the needs of your project and team. &nbsp;I see the SemVer workflow as one such adaptation. &nbsp;It just so happens that this model applies nicely to multi-versioned RESTful APIs.</p>
