---
title: 'Effortless security with OPA and Atlantis'
published: true
tags:
  - Terraform
---

Let the engineers off the leash
======
As you all know, maintaining Infrastructure-as-Code is tricky. Not only you have to keep some consistency, and implement auditing and logging, but also avoid collisions and continuous drift. While there are some pretty neat tools for this that you should definitely keep in you arsenal, not all of them offer efficient scaling and long-term control. If you practice DevOps as it was meant to be practiced (and we at GWI definitely do), then you already know that ownership of infrastructure is rather fluid. Of course, you have some distributed DevOps team to take care of innovation, internal tooling and spicy infrastructure-related problems, but what about the rest of the engineers? Front-end and back-end developers in our case participate in day-to-day DevOps tasks, such as resource provisioning via Terraform, be it related to the cloud or Kubernetes clusters using Helm provider.

This is the moment when we realized that this is perfectly doable in 20 people, but not so much in engineering teams accommodating over ~100 members. Some non-intrusive solution was desperately needed to keep track of what everyone’s doing, and Atlantis saved the day. This tool runs directly in the cluster, communicates with your Git repository, and essentially runs Terraform plan and apply for you, instead of the engineer doing it locally (e.g., without a trace). We already have a glorious blogpost about this, so feel free to do some reading before we dive deeper into the muddy waters!

In any case, Atlantis lets us maintain at least some control and oversight, while simultaneously allowing others to freely perform their daily tasks without DevOps team being “the grumpy gatekeeper” that constantly blocks others. This is all nice and dandy, but… here comes the but. How do you make sure, despite all the freedom and responsibility that is distributed among the engineering teams, that some resources are left untouched? That someone won’t just come and poof… delete the entire cluster? In the default state, Atlantis doesn’t care much about this. Sure, everything is audited in version control (and you know to whom the PR belongs, and what he did), but that doesn’t prevent someone (even with limited local privileges) to actually go and do some harm (intentional, or unintentional). In this case, Open Policy Agent comes to the rescue!

OPA — the annoying security guy who won’t leave you alone
======
Although Atlantis works wonders for us, it can always be enhanced. Up until this point, we essentially used just a couple of features that were more than plenty for our use-case — automatic plan whenever someone opens a new PR, and then the possibility to simply run atlantis apply, which performs all of the desired changes. Pretty straightforward if you can ask me. Still, the question “how do we protect our resources” remained. Then we stumbled upon Open Policy Agent, a framework for writing policies, rules, and… yes, it sounds a lot like bureaucracy. The upside is — it is a bureaucracy you only have to go through once. Have you ever been frustrated by IAM? RBAC in Kubernetes? Proprietary authorization in your microservice? Well, OPA solves all of that. It takes virtually any JSON output (from your service, Terraform, Kubernetes, what have you), and applies your own policies to it.

The good news is that you don’t need to tweak your resources in any way. The bad news is that the policies cannot be written in a regular imperative language such as Python, as OPA relies on Rego instead. This declarative language is slightly similar to HCL from HashiCorp, and resembles a “lightweight, declarative Golang”. After all, OPA was written almost entirely in Go. The syntax is pretty straightforward, and the full documentation will surely give you a nice headstart. The architecture is easy to grasp as well, as each policy is divided into complete rules (simply deny or allow statements), and partial rules (that actually evaluate your JSON). You only need to specify the input and “drill down” the JSON hierarchy until you find the resource you desire. Each policy is then packaged (you can name it however you want), and can be distributed. Those policies are then used by either a standalone OPA server interpreting communication between your services (Gatekeeper for Kubernetes, Envoy proxy for microservices), or an SDK/library/plugin for remaining use-cases.

The struggle is real (& rewarding)
------

Now that we have the formalities and introductions over with, let’s get down to business and showcase our approach. Rego took us quite some time to figure out, iron out the quirks, and evaluate the resource we actually wanted. As each rule can be parsed either standalone, or recursively, we oftentimes ran into unexpected bugs, or evaluation of completely unrelated resources. Thankfully, though, once you grasp the concept, the potential is limitless. Same rules can be applied to virtually anything, be it your IaC, Kubernetes, cloud or even internal IAM, as long as you use the OPA server/controller/plugin. You only need to tweak the input, and OPA will automatically take the JSON output (for example Terraform plan), and pass it into your policy. Once it clicks, you can code whatever you want without the need to use a vendor-specific tool (although in our case, OPA is just a nice addition).

When we finally got familiar with the syntax and language as a whole, we moved onto tests. First step was to create a mock Terraform plan to evaluate against, then create a test that would take it as an input, and check it against our policy. This is the most streamlined way to actually test your policies locally, before you start distributing them, or embedding them in your workflow. All the tests do is parse the rule, parse the input, and say whether the policy passed or failed based on your intention. We decided to follow a fixed structure to unify different policies, and make sure that the code can be well-understood by other team members and engineers. Well, creating complicating policies would be essentially an anti-thesis to what we were trying to do — simplify control and oversight, promote transparency, unblock others. And final step? Simply creating a specialized directory in our infrastructure repository dedicated just for policies.

**Handy methodology:**

  * Create an entrypoint for evaluation (assign an input to a variable)
  * Define one helper partial rule that calls the entrypoint and negates it
  * Optionally create an allow/deny list to compare your input against (depends on the policy)
  * Define partial rules with the entrypoint variable as parameter, perform operations
  * In a new partial rule, recursively call the following one (chaining them together and passing the variable forward, the same as calling functions in regular programming language)
  * Finally, create one complete deny rule that calls the helper partial rule and outputs a message in case the policy fails

This helps promote readability, and avoids complexity. I know, I know… it seems complicated. But in this case, it saved us a lot of time (and nerves). Thanks to this approach, we were able to avoid assigning an input to a variable in every single rule, or call all partial rules individually in the deny block.

Example policy that prevents a resource from being deleted:
------

```golang
package some_name

import future.keywords.in
import input as params

# List of resources
some_list = {
 "some_resource",
}

# List of denied actions
deniedActions = {"delete"}

# Helper partial rule that calls entrypoint
all_policies {
 not any_resource_match
}

# Entrypoint listing resources from input that calls the next rule
any_resource_match {
 some x in [x | x := params.resource_changes[_]; x.type in prevent_destroy]
 checkActions(x)
}

# Entrypoint listing actions from input that calls the next rule
checkActions(resource) {
 isDeniedAction(resource.change.actions)
}

# Check if the intended action is allowed
isDeniedAction(actions) {
 actionSet := {x | x := actions[_]}
 count(deniedActions & actionSet) == 1
}

# Final complete rule calling helper partial rule
deny[msg] {
 not all_policies
 msg := "You attempted to destroy a prohibited resource."
}

```
In the policy above, we extracted two objects from JSON Terraform plan — list of resources, and list of actions. We then specified allowed resources and denied actions (to evaluate against), and simply passed the variable forward and created a diff to tell us whether action “delete” is present in the array of input actions, and at the same time if the resource containing this action matches the list of allowed resources the rule should be applied to. No worries, it took us ages to get this right! And we’re still learning.

The big wins & fruitful reward
------

Thanks to everything I have described above, we were able to successfully test policies locally, evaluate correct resources, and output the right results. Everything went smoothly, hasn’t it (insert crazy happy emoji)? In any case, we managed to come up with three distinct use-cases that we were struggling with since the dawn of time:

  * Checking correct labels to ensure that billing of handpicked resources (such as GKE cluster or databases) is monitored by our team leads
  * Preventing people from destroying certain resources without DevOps approval
  * Protecting our Terraform state buckets from being modified in any way

Our use-case was simple — running policies only against Terraform resources (or rather, against the whole plan output). We haven’t delved deep into Kubernetes stuff just yet, same with our microservices. We barely scratched the surface of insane complexity (& potential) OPA offers. But to keep our sanity in check, we shall take it slow and steady!

Merging Atlantis and OPA together
======

As already mentioned in the introduction, so far we haven’t used Atlantis for anything too fancy. But the thing about Atlantis is that it’s capable of great things, massive customizability and most importantly, contains a number of plugins that are at your disposal. Such as Conftest, a handy extension that runs OPA under the hood, and runs the policies on your behalf. Technically speaking, you don’t need to have OPA tests in the first place. They just help you test things locally, but have no real effect when it comes to Atlantis. That’s where Conftest comes into play. Thanks to some wizardry behind the scenes, Atlantis can, with proper configuration, run the necessary commands without explicitly telling it what to run. But first things first.

We have modified our Atlantis values.yaml to include a repoConfig block, which allows you to further customize your workflows. Then we specified the scope (which repository that particular configuration shall be applied to), policy_sets with name of the policies (not files, but package names as described earlier) and path in the repository, policy owners that can overwrite policy evaluation using atlantis approve_policies (last resort measure to unblock someone), and finally a custom workflow that runs policy_check after init and plan (before manual apply is allowed).

```yaml
repoConfig: |
  repos:
    - id: github.com/<organization>/<repo>
      workflow: custom
  policies:
    owners:
      users:
      - <some_user>
    policy_sets:
      - name: <some_policy_package>
        path: <some_path>
        source: local
  workflows:
    custom:
      plan:
        steps:
          - init
          - plan
      policy_check:
        steps:
          - show
          - policy_check:
              extra_args: ["--all-namespaces"]
```

Beware, though — Conftest by default only evaluates against main policy package, so if you name your package differently (or have a multiple policies), it won’t work. To allow evaluation of all policies regardless of name, we had to add “ — all-namespaces” flag. Same goes for local source, as Atlantis doesn’t offer the possibility to pull remote policy packages by default. We also had to add an init command for Atlantis to check out policy folder in our repository containing individual policies, so that it can find the specified packages.

Of course, the policy folder also includes tests and mock data in JSON that we extracted from a Terraform plan, and customized. Each test then evaluates a different part of the mock data, covering all use-case scenarios. Imagine unit tests, just more straightforward, as shown in the example below. This helps us — and other engineers, naturally — test the policies locally via opa test. Most importantly, though, it verifies that the evaluation will behave as expected. Another benefit is the breakdown of silos, as each engineer can get familiar with the code and contribute, or at least check what went wrong if his Terraform plan fails. Developers can output their plan locally, test if everything is alright, and then proceed to push their changes. Atlantis will then take care of the rest!

The big wins & fruitful reward
------
```yaml
package labels

# Tests that a resource with both labels passes
test_labels_env_success {
 all_policies with input as data.mock.valid_labels
}

# Tests that a resource with missing env label fails
test_labels_env_missing_fail {
 not all_policies with input as data.mock.missing_env_label
}
```

Example of mock data:
------
```json
{
    "mock": {
        # Resource with correct labels
        "valid_labels": {  
            "planned_values": {
                "root_module": {
                    "resources": [
                        {
                            "type": "google_storage_bucket",
                            "values": {
                                "labels": {
                                    "community": "platform",
                                    "environment": "staging"
                                }
                            }
                        }
                    ]
                }
            }
        },
        # Resource with missing environment label
        "missing_env_label": { 
            "planned_values": {
                "root_module": {
                    "resources": [
                        {
                            "type": "google_storage_bucket",
                            "values": {
                                "labels": {
                                    "community": "platform"
                                }
                            }
                        }
                    ]
                }
            }
        },
```

As we have both production and testing cluster, we decided to deploy Atlantis on testing, create a mock repository with a limited scope, and see whether the ecosystem behaves as we intended. As we quickly realized, local testing is fundamentally different from live one, so a quick run-through was needed before deploying everything on production. We let Atlantis apply the custom workflow on our mock repository, pull and check out our main repo (with the policy folder), and do its magic. After testing various resources and behaviors, we concluded that it’s good to go, and went ahead to move everything to production.

Of course, even this step wasn’t painless. We accidentally introduced a bug that was triggered only in monorepo (e.g., only certain teams had this issue). Both of our policies evaluated a special input (in our case resource_changes part of JSON plan), which caused the policies to fail when a resource with wrong labels or delete action was present. Although it was completely unrelated to the PR, Terraform runs plan on all resources, hence taking in account even existing ones. After various trial & error, we decided to leave it as-is. It motivated other engineers to fix their labels and double-check that everything is in order. The same couldn’t be said for the prevent_destroy policy, so we had to introduce some changes to only evaluate resources that are actually being modified (and exempt “no-op” from restricted actions).

After ironing out similar unexpected behavior, we managed to wrap this up and officially announce the changes to the rest of engineering. As of now, everything works as required, and there are no major blockers. Quite the opposite — we incentivized others to seek our approval in case of some bigger change (such as deleting a cluster), and at the same time delegated the responsibility to them without unnecessary gatekeeping. It works very well for us, and hopefully we can take this further down the road.

Wrap up & final words
======

After the introduction of Atlantis, this was another massive learning experience for the whole team. Not only it pushed us to learn a completely new language that requires a different mindset and approach, we also closed the holes in our security posture (or rather, took our DevOps culture to the next level). We made a ton of mistakes during this whole initiative, but we strove to learn from them and most important of all — we knew that this would help the organization as a whole. Our fellow engineers were more than patient with us when we caused unintentional blocks, such as the bugs mentioned in the former section, but in the end, it was well worth it.

As of now, we host everything ourselves. Styra (the company behind OPA) offers a very advanced cloud with prepared policies for various use-cases and some neat features, same as HashiCorp with their Sentinel framework (OPA support added in Terraform v1.4). Despite all that, we decided to go with the embedded open-source version in Atlantis for the foreseeable future. We can still scale fairly easily, and until we count our policies in 100s, there is no reason to switch. Right now, we have ll of our itching use-cases covered, but we definitely strive to push forward and introduce more advanced policies when the time comes. Until then, we hope you found our journey inspiring and who knows, maybe we will release a second part in the future!