---
title: "If an Agent Can’t Find It, Neither Can Your New Hire: Codebase Structure in the Age of AI"
date: 2026-11-07 15:52:38 +0530
tags:
  - ai
  - agent
  - ai-agents
  - clean-code
  - wip
---
_The engineering habits we built for human maintainers now decide how well AI agents can find, understand, and safely change your code._

---

Somewhere in your repository there is a file everyone knows about and nobody loves. It started as a reasonable module. Then it picked up a validation helper, a pricing rule, a database query, a retry wrapper, and a formatting function that "didn’t really belong anywhere else." It is now 2,400 lines long. Everyone on the team has learned to work around it: search for the thing you need, read just enough, change it carefully, and hope.

Now hand that file to an AI coding agent and ask for a small change.

The agent has no folklore. It hasn’t been on the team for three years. It doesn’t know which parts are dead code, which parts are load-bearing, or which "temporary" function everything quietly depends on. It has what any newcomer has: file names, folder names, function signatures, imports, tests, and whatever it manages to read. From that evidence alone, it has to decide where a change belongs, what it might break, and when it has understood enough to stop looking.

That is the shift this article is about. We have always structured code so that developers could understand and maintain it. That hasn’t stopped being true. But there is now a second reader, one that navigates by search and file reads rather than by memory and hallway conversations, and that reader is influenced by structure in very direct ways. Your codebase is an interface, and agents are now one of its consumers.

None of this requires believing that agents are magic, or that they will replace anyone. It requires only a modest observation: **the things that make a codebase hard for a new teammate to work in tend to make it hard for an agent to work in too.** The good news is that the remedy is mostly the engineering discipline we already knew we should be practicing.

## 1. The hidden cost of large files

Let’s start with the obvious suspect, and then immediately defend it.

A large file with several responsibilities is hard to work with because understanding any one part of it requires wading through the rest. Suppose you ask an agent to change how order discounts are rounded, and the relevant function lives in `order_service.py`, a 1,800-line file that also contains:

- request validation
- discount and tax calculation
- SQL queries for orders and inventory
- calls to a payment provider
- email notification templates
- a handful of date-formatting helpers

To make a safe change, the agent has to figure out which of those pieces interact with the rounding logic. Does the tax calculation call the discount function? Is the rounded value stored somewhere and compared later? Does the payment code assume a specific precision? None of that is visible from the function itself. The agent has to read the surrounding file to find out, and the surrounding file is mostly irrelevant. A small request now demands an understanding of a disproportionately large amount of unrelated code, and the more unrelated code is in play, the more chances there are for a plausible but wrong inference.

Contrast that with a layout where the discount logic lives in its own focused module:

```text
orders/
  pricing/
    discounts.py
    tax.py
    rounding.py
  persistence/
    order_repository.py
  payments/
    payment_gateway.py
  notifications/
    order_confirmation_email.py
```

Now the task has a natural boundary. The agent opens `discounts.py` and `rounding.py`, sees what they import and what imports them, and can reason about the blast radius. The question "what could this change affect?" has a much smaller search space.

### File size isn’t the villain

Here is the important caveat: **the problem is not length, it is mixed responsibilities.**

A 1,500-line file that implements a single, coherent parser or a single state machine can be perfectly navigable, because every line is relevant to the same concern. Splitting it into eight files might make things worse. Meanwhile, a 150-line file that mixes HTTP handling, SQL, and business rules is already a problem. Line count is a smell detector, not a verdict. ~~We’ll come back to this in the over-engineering section.~~

The useful framing is a _reasoning boundary_. A good file is one where you can say, "to change X, I need to understand this file and its declared dependencies, and nothing else." Small, cohesive files make those boundaries cheap to find. Large, tangled ones make them invisible.

## 2. The `utils` problem

If large files are the obvious offender, the junk-drawer utility module is the sneaky one.

Every ecosystem has its version: `utils.ts`, `helpers.py`, `common.js`, `Util.java`, `lib/misc.sh`. They start innocently. Someone writes a function that two modules need, and the file becomes the default place to put anything reusable. A year later it exports sixty functions that share exactly one property: at some point, someone wanted to use them more than once.

For a human, this is annoying. For an agent, it is a discoverability failure. Consider what happens when an agent needs to format a currency amount. It can search for "currency," and if it’s lucky it finds `formatMoney` in `utils.ts`. But it might also find `formatPrice`, `toCurrencyString`, and `displayAmount`, scattered in the same file or in other `utils` files, each slightly different. Which one is canonical? Which handles negative numbers? Which respects locale? When the agent can’t tell, it does what a rushed human does: it writes a new one. Now there are five.

The junk drawer also hides what exists. A well-named module advertises its contents; a generic one forces you to read the whole thing to know whether the function you need is in there.

### Before and after

Here’s a typical "before":

```text
src/
  utils.ts              # 1,100 lines, 63 exports
  components/
  services/
  api/
```

Inside `utils.ts` you might find `formatMoney`, `parseCsvRow`, `debounce`, `isValidIban`, `slugify`, `retryWithBackoff`, `addBusinessDays`, `deepMerge`, and `hashPassword`, all next to each other with nothing in common.

And an "after":

```text
src/
  shared/
    money/
      formatMoney.ts
      parseMoney.ts
      currencyRounding.ts
    dates/
      businessDays.ts
      dateRanges.ts
    text/
      slugify.ts
      truncate.ts
    resilience/
      retryWithBackoff.ts
    security/
      passwordHashing.ts
  validation/
    iban.ts
    email.ts
```

The same code is now organized by _what it is about_. If the task mentions money, the agent (or the human) goes to `shared/money/`. The folder name does the first stage of search for free.

### Group by responsibility, not by reusability

The distinction that matters is _why_ things share a file. "These functions are reusable" is not a responsibility. It is a property that nearly all well-written functions have, which is why it makes such a poor organizing principle. "These functions all deal with money formatting and rounding" is a responsibility.

Note what this does _not_ mean: it does not mean one function per file. `money/` above could just as reasonably be a single `money.ts` containing `formatMoney`, `parseMoney`, and the rounding helpers, as long as they are cohesive and the file stays readable. Related functions are happier together: they share types, they share invariants, and a reader (human or otherwise) benefits from seeing them side by side. The goal is a file whose name tells you accurately what is inside, not a file for every function.

## 3. Context is a limited resource

It helps to understand, at a high level, how agents actually work on code. An agent operates within a context window: a bounded amount of text it can consider at once. Everything it is reasoning about, including your request, the files it has read, search results, tool output, and its own earlier steps, has to fit in that window. And even when it fits, attention isn’t uniform. Irrelevant material is not merely wasted space; it competes with the relevant material for influence over what the agent does next.

This leads to a counterintuitive point: **more context is not automatically better.** If you hand an agent forty files "just in case," you haven’t made it better informed; you’ve made the signal harder to find. The same is true of a codebase that forces exploration. When the structure doesn’t tell the agent where to look, it compensates by reading widely: opening files that turn out to be unrelated, following imports that lead nowhere, pulling in code that has nothing to do with the task. Each of those detours adds noise.

The practical lesson is that _relevant_ context beats _maximal_ context. A well-organized codebase makes relevant context easy to assemble: the right file is named in a way you can find, its dependencies are explicit, and the code in it is about one thing. A poorly organized one makes you, or the agent, guess.

A word of caution about framing, though. It’s tempting to reduce this whole topic to token budgets, and that would be a mistake. Context limits are one reason structure matters, but I wouldn’t put a number on the savings, and I’d be skeptical of anyone who does; it varies enormously with the task, the tooling, and the model. The deeper benefits are about discoverability (can the agent find the right place?), understandability (can it tell what the code does?), and safe changeability (can it modify the code without breaking something unrelated?). Those are the same benefits we’ve always wanted. Context efficiency is a side effect, not the goal.

## 4. Separation of concerns, across the stack

Separation of concerns is the oldest idea in this article, and it tells an agent something specific: _this is where a change of this kind belongs._ When business rules, presentation, data access, external integrations, and infrastructure are tangled together, every change request becomes a puzzle about which layer you’re really touching. When they’re separated, the request usually maps cleanly onto one place.

This is true everywhere, not just in web apps.

**Frontend.** A React component that fetches data, transforms it, manages form state, and renders markup is four jobs in one function. Compare:

```text
features/checkout/
  CheckoutForm.tsx          # rendering and user interaction
  useCheckoutState.ts       # state transitions
  checkoutApi.ts            # network calls
  calculateOrderTotals.ts   # pure business logic
```

Ask for "change how shipping is calculated" and the target is obvious. It is a pure function with tests, not something buried in a `useEffect`.

**Backend.** In a Django or Spring service, the classic split of views/controllers, services, and repositories exists for this reason. If a view function contains an ORM query, a pricing rule, and a call to a payment API, then "add a new discount type" forces the reader to sort out which lines are policy and which are plumbing.

```python
# Hard to change safely: policy, persistence, and integration interleaved
def checkout(request):
    cart = Cart.objects.filter(user=request.user).prefetch_related("items").first()
    total = sum(i.price * i.qty for i in cart.items.all())
    if request.user.is_member and total > 100:
        total *= 0.9
    stripe.Charge.create(amount=int(total * 100), source=request.POST["token"])
    ...

# Easier: each concern has an obvious home
def checkout(request):
    cart = cart_repository.get_for_user(request.user)
    total = pricing.calculate_total(cart, request.user)
    payment_gateway.charge(total, token=request.POST["token"])
    ...
```

**Databases.** A migration that alters a schema, backfills data, and drops a legacy column in one script combines changes with very different risk profiles. Splitting schema changes from data backfills from cleanup makes each step reviewable and reversible. The same goes for SQL: a 400-line query that joins eight tables, applies business rules, and formats output for a report is much easier to change as a set of named CTEs or views, each doing one understandable thing.

**Infrastructure.** A single Terraform file declaring networking, IAM, databases, and application services forces every infrastructure change to be reasoned about against everything else. Organizing by module (`network/`, `database/`, `iam/`, `app_service/`) with explicit inputs and outputs means a request like "restrict access to the database subnet" has a bounded scope. The same logic applies to a Kubernetes repo split by workload and environment instead of one giant manifest, or a CI pipeline where build, test, and deploy stages are defined separately rather than as one 600-line YAML file.

**Data and automation.** A pipeline script that extracts from an API, cleans the data, applies business transformations, and loads it into a warehouse is hard to modify without risking all four. Separating `extract`, `transform`, and `load` steps (and keeping transformations as pure functions) lets you change a cleaning rule without worrying about API pagination or warehouse credentials.

In every one of these cases, the pattern is the same: clear responsibilities mean the request "change X" has an unambiguous answer to "where?"

## 5. Naming is a form of documentation

Agents navigate heavily by names. They search for them, read them in directory listings, and use them to decide which file is worth opening. Names are, in effect, the first layer of documentation, and the only layer that is guaranteed to be seen before anything else is read.

Compare:

|   |   |
|---|---|
|Generic|Descriptive|
|`utils.ts`|`shared/money/formatMoney.ts`|
|`manager.py`|`subscription_renewal_scheduler.py`|
|`processData()`|`normalizeCustomerAddresses()`|
|`handler.js`|`stripeWebhookHandler.js`|
|`data.sql`|`monthly_active_users_by_region.sql`|
|`deploy.sh`|`rollout_api_to_staging.sh`|

The generic names aren’t just uninformative. They’re _ambiguous_, which is worse. A file called `manager.py` could manage anything; to find out what it manages, you have to open it. And once you’ve opened it, you’ve spent effort on something that may not be relevant. Multiply that by every vague name in a codebase and you get the exploration overhead that makes agents (and new hires) slow and error-prone.

Good names do three things. They let a search for a domain concept ("renewal," "refund," "webhook") land on the right place. They let you rule files in or out from the directory listing alone. And they reduce the chance of duplication, because it’s obvious what already exists.

This extends beyond files. Function names should describe behavior (`calculateProratedRefund`, not `handleRefund`). Variables should carry meaning (`retryDelaySeconds`, not `d`). Modules and packages should be named for the capability they provide. The specific-over-generic rule is the same at every level.

## 6. Splitting files isn’t the same as good architecture

Here’s where it gets subtle. You can split a 3,000-line file into twenty small ones and end up with something _worse_, if all of those files reach into each other.

Tight coupling is the real enemy of safe change. If `invoice.py` imports internals from `customer.py`, which imports from `billing_rules.py`, which calls back into `invoice.py`, then changing any one of them can ripple through all of them. The files are small, but the reasoning boundary is still the entire cluster. An agent making a "targeted" edit has no reliable way to know how far the effects reach, and neither does a human reviewer.

What creates real boundaries is not file layout but **explicit contracts**:

- **Interfaces and protocols** that define what a module promises, independent of how it does it.
- **Dependency injection**, so a component receives what it needs rather than reaching out and grabbing it.
- **Narrow public APIs**, so other modules depend on a small surface instead of internals.
- **Typed inputs and outputs** that make the contract machine-checkable.

A small example in Python:

```python
class PaymentGateway(Protocol):
    def charge(self, amount: Money, token: str) -> ChargeResult: ...

class CheckoutService:
    def __init__(self, gateway: PaymentGateway, pricing: PricingPolicy):
        self._gateway = gateway
        self._pricing = pricing

    def complete(self, cart: Cart, token: str) -> Receipt:
        total = self._pricing.total_for(cart)
        result = self._gateway.charge(total, token)
        return Receipt.from_charge(result)
```

The agent modifying `CheckoutService` can see exactly what it depends on, and can see what it _doesn’t_. It doesn’t need to understand Stripe’s SDK to change how receipts are built. And because the dependencies are injected, tests can substitute a fake gateway, which gives the agent (and you) a fast way to verify changes.

The same principle shows up elsewhere: Terraform modules with declared variables and outputs rather than shared global state; Java packages exposing interfaces rather than concrete classes; React features that export one public component rather than every internal hook; SQL views that serve as stable contracts between raw tables and reports. The shared idea is that a boundary should be something you can _state_, not something you have to infer.

## 7. Consistency and predictability

Every codebase has conventions, whether or not anyone wrote them down. The question is whether they’re _consistent_. When every service follows the same layout, an agent that has seen one has effectively seen them all. Where does the data access live? Where do the tests go? What do the entry points look like? The answers are predictable, so navigation becomes cheap.

```text
services/
  billing/
    api/
    domain/
    persistence/
    tests/
  notifications/
    api/
    domain/
    persistence/
    tests/
```

Predictability means less guessing. If `billing/` and `notifications/` are laid out the same way, then "where’s the repository for notifications?" has an obvious answer. Contrast that with a codebase where one service uses layered folders, another is organized by feature, a third dumps everything in a single directory, and a fourth has three competing patterns because it was refactored halfway. Every module requires fresh orientation. Inconsistent naming adds to this: if one module says `get_user`, another `fetchUser`, and a third `loadUserRecord` for the same concept, search stops being reliable.

But consistency has limits, and it’s worth being careful here. **Consistency doesn’t mean forcing every technology or domain into the same architecture.** A Terraform repository doesn’t need to look like a React app. A data pipeline doesn’t need hexagonal architecture. A small CLI script doesn’t need a service layer. The aim is consistency _within_ a domain, and clear conventions _across_ them. A short `CONTRIBUTING.md` or per-directory README describing the local conventions is often worth more than any amount of structural uniformity, and it’s the kind of documentation both humans and agents actually read.

## 8. Don’t over-engineer it

Now for the counterweight, because this advice is easy to take too far.

If some separation is good, it’s tempting to conclude that maximum separation is best: one function per file, an interface for every class, a factory for every constructor, a layer for every concern. That path leads somewhere unpleasant. When a simple operation is spread across eleven files and four abstraction layers, you’ve traded one navigation problem for another. To understand a single behavior, you have to chase it through indirection that exists only because someone wanted the structure to look clean.

Excessive fragmentation has real costs:

- **Lost locality.** Related code that belongs together gets scattered, so understanding a feature means opening many tiny files.
- **Indirection without payoff.** Interfaces with one implementation and no foreseeable second one add ceremony but not flexibility.
- **Harder navigation.** Hundreds of two-line files make directory listings long and names hard to keep distinct.
- **More surface for inconsistency.** The more pieces there are, the more places conventions can drift.

The goal is _cohesive responsibilities_, not small files. If three functions change together, are understood together, and share the same types, put them in the same file. If a module is 900 lines but has one clear purpose and a clean internal structure, leave it alone. Don’t split code just because a tool or a style guide flagged a number.

This is why I’d treat file size as a **signal, not a rule.** A file growing past a few hundred lines is a prompt to ask, "Is this still about one thing?" Sometimes the answer is yes, and the right move is to do nothing. Sometimes the answer is that there are three responsibilities hiding in there, and splitting along those lines will help. Either way, the question is about responsibility, not line count. There is no universal threshold, and any article, linter, or team that claims otherwise is optimizing for the wrong thing.

A useful test: _if someone asks for a change to X, how many files must you read to be confident you understand X?_ If the answer is one or two, you’re probably in good shape. If it’s ten, you may be over-fragmented. If it’s one giant file plus a lot of guessing, you’re probably under-separated.

## 9. Practical recommendations

If you want a codebase that works well for agents, and, not coincidentally, for new teammates, here is what I’d actually do.

**Keep files focused on one responsibility.** Not one function, one _reason to change_. Use size as a nudge to look closer, not as a rule to obey.

**Adopt predictable folder structures.** Decide where things live in each part of the system and stick to it. Make the structure answer "where would I look for X?" before anyone has to ask.

**Name things descriptively.** Prefer `subscription_renewal_scheduler.py` to `manager.py`. Be specific in file, function, module, and variable names, and delete or rename the vague ones as you touch them.

**Separate concerns by layer and by risk.** Keep business rules away from I/O, schema changes away from data backfills, infrastructure modules away from application config. Let a change request map onto one place.

**Make dependencies explicit.** Use interfaces, constructor injection, typed signatures, and narrow public APIs. A module’s dependencies should be visible at the top of the file, not discovered at runtime.

**Be consistent within each domain.** Use the same layout and naming for similar components, and write down local conventions where they aren’t obvious. Don’t force identical architecture across unrelated technologies.

**Put tests around meaningful boundaries.** Tests at module and contract boundaries give agents (and humans) a fast way to check that a change didn’t break something. They also act as executable documentation of intended behavior. Prefer tests that describe behavior over tests that mirror implementation.

**Document the map, not just the territory.** A short architecture note, a README per major directory, or an agent-oriented instructions file (many tools support one) describing layout, conventions, and how to run tests can save a lot of wandering. Keep it brief and keep it true.

**Resist unnecessary abstraction and fragmentation.** Don’t add layers, interfaces, or files until a real need shows up. Refactor toward structure when the pain appears, not in anticipation of it.

**Refactor opportunistically.** You don’t need a heroic rewrite. When you or an agent touch a muddled area, leave it a little clearer: rename an ambiguous function, extract one cohesive module, split the junk drawer along one clean seam.

None of this is new. Every item above appears in decades-old advice on maintainable software. What’s changed is the audience.

## The bigger picture

There’s a temptation to treat AI agents as a reason to throw out accumulated engineering wisdom. After all, a sufficiently capable model might just read the whole mess and figure it out. Sometimes it will. But "figure it out" is doing a lot of work in that sentence, and the cost of figuring it out is paid in time, in errors, and in the subtle bugs that come from changing something you only half understood. Those are the same costs that tangled code has always imposed on people.

What’s interesting about this moment is that it makes the value of good structure _measurable in a new way_. When a teammate struggles in a messy module, the cost is diffuse and easy to ignore. When an agent flails there, wanders through irrelevant files, or confidently edits the wrong layer, the problem shows up immediately in your workflow. The codebase is giving you feedback, and it’s feedback worth listening to, because the fix helps everyone.

And it’s worth being clear about what this is not. It is not a call to restructure your repository for the sake of a tool. Tools will change; models will get better at navigating messy code. The structure that serves agents well is structure that serves _people_ well: clear boundaries, honest names, explicit dependencies, sensible consistency. Agents have simply made the return on that investment easier to see.

## Conclusion

We used to structure code primarily to make it easier for developers to understand and maintain. Now, we also need to consider how efficiently AI agents can navigate, reason about, and modify that code.

That’s not a replacement for the old goal. It’s an extension of it. The principles are the ones we already knew: cohesion, clear responsibilities, meaningful names, explicit contracts, restraint. What’s new is that there are now two kinds of readers, and both of them start from the same place: a folder tree, a few file names, and whatever the structure gives away.

A well-structured codebase is an interface. For the developer joining next month, and for the agent opening the repo for the first time today, the design of that interface determines how quickly they can find the right place, how well they understand what they find, and how safely they can change it.

Design it accordingly.

---

<p style="text-align: center;">fin.</p>