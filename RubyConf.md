### Responsibility Nuremberg and Krishan
- Oppenheimer regretted his decision in creating his invention
  - "I am become Death, the destroyer of worlds"
- As software engineers or even as a creator it is our responsibility to decline participation in the creation of malicious software
  - e.g. google walk out


### Designing an Engineering Team
- "High bar" of highering should refer to inherit ability to learn
  - growth is linear
- Theres no bar for engineering
  - Lots of skills in ruby knowledge tree
- Interviewers often look for overlap between two people (talent bias) to measure adequacy in a skill
- Instead, we should look for diversity in team chemistry
- An example of good team attributes/roles
  - People leadership
    - High level concepts
  - Technical Leadership
  - Customer Focus and Metrics
    - Feedback from other teams and internal tools and dashboards
    - Meaningful data during deploys
  - Students
    - someone in a heavy learning position
    - teaching hospitals outperform other hospitals
      - Paradox of expertise
        - Learning new technology and keeping up to date with technology is key
- Having someone be more of an expert in one of each category is crucial to the success of a team
  - Higher based on what your team lacks
  - For planning and supporting engineer career development: https://github.com/Medium/snowflake

### Yes, You Should Provide a Client Library for Your API
- API Client Library
  - ruby gem / set of codes to do something
- Maintenance of client libraries -- documentation, naming, versioning, mapping endpoints
- API issues deals with Auth, Retry, Caching, Quotas, error handling, pagination, instrumentation etc.
- APIs are much greater than HTTP requests
- Goal of the client: Knowing the protocol

- e.g. of a Client example,
```
  client = Client.new(params identity/ auth/ etc)
  translation = client.translate("hello" to: "ja")

  puts translatin.text
```

- only includes what the user cares about
  - no URLS JSON HTTP status, retry, etc.
    - this contributes to a better developer experience

- use Ruby abstractions
  - integrating the standards with the library
  - using ruby's logger class and timeclass
  - using ActiveRecord
  - handle errors for you users
    - move them to exceptions so they don't ahve to look up errors
    - improve safety/security and sanitize via active record
    - improve performance and help with optimization caching and batching
    - provide implementation by logging all errors for your user to retrieve

- Interface Description Language (IDL)
  - machine readable specification description
    - what calls are available,
    - parameters need to be passed,
    - data types used,
    - properties like auth,
    - documentation
  - e.g. openAPI

  - metaprogramming -- dynamically generated classes is not the solution
  - code generation creates generated ruby files and classes with associated documentation.

- Getting started:
  - choose a spec standard
  - write an api descriptions
  - invoke an open source generator
  - https://daniel-azuma.com/rubyconf2018

### Uncoupling systems
- Resources on migrating and refactoring systems
- Practical object-oriented design in Ruby
- managing dependencies
- Resources:
  - Distributed Systems by Steen and Tanenbaum
  - Designing Data-intensive Applications by Martin Kleppmann
- don't be afraid to duplicate data in the form of caching
  - creating instance, creating a record, inserting into redis
    - Determine SLAs needed to be met in terms of consistency
  - communicate through message queues
  - retryable, idempotent workers that can be done async or parallel, transparent with errors
  - pre-compute works
  - identity derived database
  - determine consistency codes

- building blocks
  - workers
  - queues
  - materialized views

- Partitioning data by a cardinal value, clients
- wrapping every table in an api means no transactions and no joins
  - keep transactions and use event logs to transactionally update things
    - still has no join and now waits on previous jobs

### Building for Gracious Failure
- we can't fix what we can't see
- managing failure through visibility
  - bugstack: first add visibility on what bugs are occurring and how often
  - job success failures codes starting finishing

- gracious failures
  - returning what we can vs breaking 500
    - returning some information vs none
  - accepting what we can
    - partial exception vs bundled data
  - creating systems that are tolerant and tolerable
  - build resiliently to handle cases that you don't expect (trust carefully)
    - don't just pass errors or 500s up the chain

- assume failure is the reality where this is expected

### Let's Subclass Hash - what's the worst that could happen?
- slides available: https://michaeljherold.com/2018/11/14/rubyconf-2018-lets-subclass-hash/
- indifferent access between strings or symbols
  - subclassing hash by including multiple libraries
  - a subclass that takes Hash as a parent has to redefine all ~178 original hash methods

- mash keys is a json object method accessor
- if a hash has a method name that is the same as the key it'll call the method
  - overriding methods busts the ruby cache.

- dash (declarative hash) where you have properties like a class
- when parsing JSON there are other options like `JSON.parse`
- Debugging can sometimes be helpful to look at the instructions that make up a method
  - `RubyVM::InstructionSequence.of(method(:foo))``

### What poker can teach us about post-mortems
- slides available: https://chamblin.net/rubyconf18
- Thought: post-mortems make people risk averse
- having critical follow up items is not a great approach
- bad outcomes != bad decisions
- good decisions create bad outcomes
  - you can't do something else to prevent this
  - lesson is you should not do this again
- results oriented thinking is based on outcomes and an illusion of control
- probabilistic mapping is more realistic.
- results oriented results in avoidance of taking on new projects and new risks
- post mortems are results oriented

- questions for post-mortems to avoid bad results
- should we have prevented this? post mortems are under the assumption that all problems are worth preventing
  - you can't look back retrospectively and apply new solutions to past information
- should we prevent this in the future?
- did we understand this risk up-front?
- how should our decision making change in the future?
  - it doesn't always have to
  - you can do everything right and mitigated against clear failure cases, but can still run into faults

- Life lessons from poker
  - Tilt
  - Failure -- reviewing mistakes
  - risk mitigation
- Further courses in poker
  - udemy.com/crush-online-micro-stakes-poker

### Reducing memory usage in Ruby
- slides available: https://tenderlovemaking.com/2018/01/23/reducing-memory-usage-in-ruby.html
- Two patches
- loaded features cache
- direct ISeq Marking

- Finding memory usage
  - malloc stack tracing
- allocated memory to
  - GC
    - ObjectSpace
    - allocation_tracer (gem)
  - Malloc
    - malloc stack logging available on macosx
    - MallocStackLoggingNoCompact=1 \
    RAILS_ENV=production \
    bin/rails r `p $$; GC.start; $stdin.getc`
      - enable the logger,
      - pring the pid

    - malloc_history [PID] -allEvents > malloc_log.log
  - who calls malloc?

- Loaded Feature Cache
  - shared string optimization
    - only works for strings that are subsets till the end of the string (null terminated)
  - only loading files once

- Ruby VM
  - stack based VM
    - list of instructions and a stack
  - compiled code goes through process phases
  - source code text => ast => linked list => byte code
    - consists of parsing and compiling
    - linked list phase has optimizations for unused code etc
    - ast abstract syntax tree data structure
    - converted through recursively walking the tree

  - iseq marking
    - using frozen strings to allocate two strings and only allocate two strings of compile time and one at run time
    - byte code is stored on instruction sequences
    - iseq = instructino sequence which stores pointer to ruby objects

    - for GC
      - before 2.6 mark array iseq also has a mark array that has pointers to all ruby objects
      - duplicated information: two references to the same object to keep objects alive
      - array bloat
        - unused memory sinee arrays doubles in size
        - instead we can remove the array itself
          - by marking the object while the VM executes instructions
          - this is important because GC array lives forever and by getting rid of it we reduce a lot of memory usage

### Code reviews
- Active verb for commit messages
- taking notes
- keepings gems in abc
- IRC message
- Asking why questions
- Early architecture review

### Cache is king
- ActiveModel Serializers and vulnerability
- Instead of single calls to DB make one: Bulk Serialization
  - decreases database hits
  - indexing requires chat to redis you can cache

- Key takeaways:
  - Process in bulk
  - ruby local hash cache is faster than redis
  - framework /gem cache understand active record

  - avoid worthless database calls -- database guards
    - nil check before executing
    - `.none` on active record when checking empty array
      - `User.none.active.tall.single`  vs `User.where(:id => []).active.tall.single`
    - especially useful in reports

  - Remove datastore hits period
  - Resque workers operates on top of redis
    - dropping connection lost since redis has to heck the number of workers are currenly working on the queue using redis.

  - every datastore hit counts

### Practical guide to benchmarking your optimizations
- Benchmark
- run the old code and the new code measure time and compare

- Simple random sample
  - each element has an equal change of being chosen
    - eliminates bias

- Stratified samples
  - Divide into groups based on something...
  - Ensures we have representation from each group

- Cluster Sampling
  - divide sample into clusters
  - cheaper / less expensive

- Systematic random sampling
  - pick every nth element

- Multi-stage
  - mix and match the above

- Use strata for known variables for benchmarking every code path
- Random sampling, accounting for unknown variables
- Clusters: check how each cluster compares to the population (diffferent code paths)

- gotchas: watch out for initialization (run twice with bmbm)

- Redflags for optimization
  - loops e.g. each map
  - recursion
  - requiring a lot of Thing
  - callbacks and observers

- When should you benchmark?
  - large cron jobs
  - changes to pieces of code that are frequently run
  - code that is already slow

### No Title Required; How Leadership Can Come From Anywhere
- leadership is more than money and fame
- it's a personal journey
  - it's about people not code
- it's about the how and not just the what  
- pushing for improvement throughout the organization
- leadership: is the continuous practice of positive influence
- Flywheel
  - identify the problem
  - fix the problem
  - prevent similar classes of problems
  - teach others how to avoid the problem
  - repeat
- doing things outside of your domain that can be impactful

- Observations of leadership
- title != leadership. think about why you want that title
- incentives and values
- You can't be passive in your leadership
  - "Someone will notice my good work"
  - people are often busy with their own work
  - call out where you are making a difference
  - the best leaders are the best advocates for themselves
  - don't get stuck doing busy work

- Do these things
  - Don't talk, listen
  - self assessment
    - what are you good at
    - what would you like to improve
      - long return goals: introspective every month
        - discuss your career goals, how you'd like to grow, what you'd like to learn
        - revisit action items
  - Feedback
    - building trust from the beginning
  - dive deep
    - start from the top of customer needs to spec details down to implementation
    - dissolve and write down and share assumptions
  - shipping a successful project has many leadership impacts
    - project management
    - launch plan_name
    - stakehold communications
    - metrics from https://xkcd.com/1205

- these are all in a flywheel feedback -> dive deep -> self assess -> feedback
- mentorship
- don't ignore physical and mental health
- leadership isn't zero sum
  - just start with one improvement process or encouraging coworkers

- Resources
  - measure what matters
  - high output management
  - the power or habit
  - feedback for engineers
  - software lead weekly

### The New Manager's Toolkit
- Problems in management often occur because of a trust problem
- tool 1: one on one meetings
  - show up with questions but you need a sense of career progress
- The manager's Path by Camille Fournier
- Clear interview process examples with questions and exercises
- creating engineering ladder
- don't forget about accountability
- performance improvement plans
- crucial confrontations
  - Radical Candor by Kim Scott
  - laying out expectations with clarity and follow up
- Specific responsibilities of a manager vary widely
- A manager's job is to be available
  - office hours and active listening
  - listening means following up
    - personal kanban boards (trello)
- Interpersonal conflict
  - your job is to find the source issue and fix it first
    - usually a trust issue
    - crucial conversations
      - having a meeting to teach them how to build trust and conflict resolution
      - set up common goals
- wikis and documentation tools
- culture change processes
  - creating a shared purpose
- measure what matters (OKRs)
- start/stop/continue retrospective
- *don't forget about yourself*
- The power of vulnerability by brene brown
- Ask your manager for help
- Delegation frameworks
- Podcasts about holistic leadership (Reboot)
- Breathing Exercises
- Exercise and Meditation
- not the job
  - save everyone
  - be all knowing
- is job
  - be available
  - listen guide and coach
  - unblock and support
  - build bridges of trust
  - find connect people to the right resources
- gratitude and journaling
- podcast: managing up

### Eiffel Tower
- Make friends
- Tell stories
   - your boss isn't listening as closely as you think they might be

- Cooperation
- You can negotiate anything herb cohen
  - Understanding needs (including your own)
- Blocking enough noise but just enough so that junior engineers have exposure and context

### Resources for being part of the ruby conference
- ruby for good
- annie cannons





-
