# Supervision

#### (or how to successfully embrace failure)

---

### About...

* Worked in a variety of areas
  - Distributed systems, Robotics, Augmented Reality, High-frequency trading

* Long-term Erlang fan
  - Excited to see adoption grow through Elixir

Notes:
Not a web-guy, not so much experience with Elixir

---

### The Pillars of Robustness

1. Pre-emptive scheduling

2. Process isolation

3. Supervision

Notes:
PE allows progress to be made. Isolation limits the effect of faults. The BEAM VM takes a unique approach to fault handling.

---

### Some History

* Erlang inherited many design goals from PLEX/AXE

* Designed primarly for fault-tolerance

* Error-handling was an explicit feature

Notes:
Stringent reliability requirements (fines etc), fault-tolerance to both hw failures
and sw-errors.

---

### Architecture

Supervision is a pattern built on primitives.

<svg width="640" height="300" xmlns="http://www.w3.org/2000/svg" xmlns:svg="http://www.w3.org/2000/svg">
 <g>
   <rect stroke="#000000" id="svg_13" height="136" width="328" y="19" x="96" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" fill="#ffffaa"/>
  <rect stroke="#000000" fill="#aad4ff" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x="96" y="225" width="328" height="70" id="svg_1"/>
  <text fill="#000000" stroke="#000000" stroke-width="0" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x="256" y="268" id="svg_3" font-size="24" font-family="Sans-serif" text-anchor="middle" xml:space="preserve">BEAM Virtual Machine</text>
  <rect stroke="#000000" fill="#aaffaa" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x="96" y="155" width="328" height="70" id="svg_4"/>
  <text fill="#000000" stroke="#000000" stroke-width="0" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x="258" y="198" id="svg_7" font-size="24" font-family="Sans-serif" text-anchor="middle" xml:space="preserve">kernel</text>
  <rect stroke="#000000" fill="#d4aaff" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x="250" y="85" width="174" height="70" id="svg_8"/>
  <text fill="#000000" stroke="#000000" stroke-width="0" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x="336" y="128" id="svg_9" font-size="24" font-family="Sans-serif" text-anchor="middle" xml:space="preserve">stdlib</text>
  <text stroke="#000000" transform="matrix(2.1736786365509033,0,0,4.320018437051345,-267.94286270067096,-1252.6213294561146) " xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="24" id="svg_2" y="349" x="326.71663" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="0" fill="#f0f0f0">}</text>
  <text id="svg_11" stroke="#000000" transform="matrix(2.173678636550903,0,0,2.6502194210068057,-268.94286270067096,-730.528199648713) " xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="24" y="326.84068" x="327.7604" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="0" fill="#f0f0f0">}</text>
  <text xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="24" id="svg_12" y="126" x="510" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="0" stroke="#000000" fill="#f0f0f0">Patterns</text>
  <text xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="24" id="svg_14" y="231" x="520" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="0" stroke="#000000" fill="#f0f0f0">Primitives</text>
  <text style="cursor: move;" id="svg_17" fill="#000000" stroke="#000000" stroke-width="0" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x="253.00781" y="60" font-size="24" font-family="Sans-serif" text-anchor="middle" xml:space="preserve">other applications</text>
 </g>
</svg>

* Primitives - links and monitors
* Patterns - OTP supervision

Notes:
The primitives are links and monitors and the pattern is encapsulated by gen_supervisor.

---

### Links

* Based on the electro-mechanical 'C-wire'

* Bi-directional connections between processes
  * Termination sends exit _signal_ to linked processes
  * The signal carries the termination reason
  * If reason != :normal, linked processes terminate

* See BIFs :erlang.link/1, :erlang.spawn_link/1,2,3,4

Notes:
Idea credited to Mike Williams, who along with a guy called Bjarne Dacker founded the CSL
at Ericsson, and who wrote the first Erlang VM. Termination propagates by default causing
all linked proceses to exit. Only one-link per-process pair and not ref-counted. In OTP,
rare to use these bifs directly.

---

### Trapping Exits

* Can change the behaviour of signal receipt
  * :erlang.process_flag(:trap_exit, true)

* Exit signals get converted to messages
  * {:EXIT, fromPid, reason}

Notes:
Now we can deal with failure. Cannot trap the reason 'kill'.

---

### Monitors

* Sometimes links are too 'strong'
  * If the observer exits, the subject exits.

* Monitors enable 'weak' observation
  * Multiple monitors can be established per pair
  * ref = :erlang.monitor(:process, pid)

* Message received when subject exits
  * {:DOWN, ref, pid, reason}

Notes:

---

### Ignore the last 3 slides

(if you're writing OTP-compliant applications)

Notes:
If you use the OTP framework, you won't need to use links and monitors directly. But you will
need to understand OTP Supervisors

---

### OTP Supervisors

* A supervisor is a process that:
  * Starts (and links) its child processes
  * Restarts them when the fail

* Encapsulated in its own behaviour module
  * Your code implements only :init/1 and returns
    * The restart strategy
    * The list of children to start/restart

Notes:
We can probably see that supervisors link their children and trap-exits in order to monitor
them.

---

### Supervision trees

<svg width="640" height="480" xmlns="http://www.w3.org/2000/svg" xmlns:svg="http://www.w3.org/2000/svg">
 <!-- Created with SVG-edit - http://svg-edit.googlecode.com/ -->
 <g>
  <title>Layer 1</title>
  <circle fill="#aad4ff" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" cx="321" cy="208.75" r="34.66987" id="svg_13"/>
  <g id="svg_14">
   <circle fill="#d4ffaa" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" cx="186.99998" cy="208.75" r="34.66987" id="svg_7"/>
   <circle fill="#d4ffaa" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" cx="187.49997" cy="208.25" r="22.16987" stroke="#e5e5e5" id="svg_8"/>
  </g>
  <g id="svg_15">
   <circle fill="#d4ffaa" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" cx="321" cy="75" r="34.66987" id="svg_16"/>
   <circle fill="#d4ffaa" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" cx="321.49998" cy="74.5" r="22.16987" stroke="#e5e5e5" id="svg_17"/>
  </g>
  <circle fill="#aad4ff" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" cx="462.99998" cy="208.75" r="34.66987" id="svg_18"/>
  <circle fill="#aad4ff" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" cx="109.99999" cy="349" r="34.66987" id="svg_19"/>
  <circle fill="#aad4ff" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" cx="251.99997" cy="349" r="34.66987" id="svg_20"/>
  <line fill="none" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x1="125" y1="317" x2="172" y2="240" id="svg_21"/>
  <line fill="none" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x1="205" y1="240" x2="246" y2="312" id="svg_22" stroke="#e5e5e5"/>
  <line fill="none" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x1="208" y1="180" x2="297" y2="100" id="svg_23"/>
  <line fill="none" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x1="322" y1="111" x2="324" y2="171" id="svg_24"/>
  <line fill="none" stroke="#e5e5e5" stroke-width="5" stroke-dasharray="null" stroke-linejoin="null" stroke-linecap="null" x1="350" y1="98" x2="440" y2="180" id="svg_25"/>
 </g>
</svg>

Notes:

---

### Child specifications

* _Which_ children are to be started
  * An MFA triple - {module, :start_link, args}

* _When_ to restart each child
  * :permanent - always restart
  * :transient - restart only on _abnormal_ termination
  * :temporary - never restart

* Other stuff

---

### Static Strategies

* One-for-one
  * Restart only the child that fails.

* One-for-all
  * Restart every child if any fails.

* Rest-for-one
  * Restart the failing child and any subsequent children

---

### Dynamic Strategies

* Simple one-for-one
  * The specification contains a _template_
  * Children only created on demand
    * Supervisor.start_child/2

* Often supervise non-recoverable resources
  * E.g. tcp connections

---

### Restart frequency

* An upper-limit on the restarts/interval
  * Defined as _p_ restarts in _q_ seconds

* Exceeding this causes the supervisor to terminate
  * _Its_ supervisor will then takeover

* A restart frequency, _not_ a failure frequency
  * E.g. temporary children do not affect this

---

### Example

<svg width="640" height="420" xmlns="http://www.w3.org/2000/svg" xmlns:svg="http://www.w3.org/2000/svg">
 <!-- Created with SVG-edit - http://svg-edit.googlecode.com/ -->
 <g>
  <title>Layer 1</title>
  <circle id="svg_13" r="34.66987" cy="208.75" cx="321" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="#aad4ff"/>
  <g id="svg_14">
   <circle id="svg_7" r="34.66987" cy="208.75" cx="186.99998" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="#d4ffaa"/>
   <circle id="svg_8" stroke="#e5e5e5" r="22.16987" cy="208.25" cx="187.49997" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" fill="#d4ffaa"/>
  </g>
  <g id="svg_15">
   <circle id="svg_16" r="34.66987" cy="77" cx="321" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="#d4ffaa"/>
   <circle id="svg_17" stroke="#e5e5e5" r="22.16987" cy="76.5" cx="321.49998" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" fill="#d4ffaa"/>
  </g>
  <circle id="svg_18" r="34.66987" cy="208.75" cx="462.99998" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="#aad4ff"/>
  <circle id="svg_19" r="34.66987" cy="349" cx="109.99999" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="2,2" stroke-width="5" stroke="#e5e5e5" fill="#aad4ff"/>
  <circle id="svg_20" r="34.66987" cy="349" cx="251.99997" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="2,2" stroke-width="5" stroke="#e5e5e5" fill="#aad4ff"/>
  <line id="svg_21" y2="240" x2="172" y1="317" x1="125" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="none"/>
  <line stroke="#e5e5e5" id="svg_22" y2="312" x2="246" y1="240" x1="205" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" fill="none"/>
  <line id="svg_23" y2="100" x2="297" y1="180" x1="208" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="none"/>
  <line id="svg_24" y2="171" x2="324" y1="111" x1="322" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="none"/>
  <line id="svg_25" y2="180" x2="440" y1="98" x1="350" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="none"/>
  <text xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="20" id="svg_2" y="160" x="124" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="2,2" stroke-width="0" stroke="#e5e5e5" fill="#e5e5e5">Task Supervisor</text>
  <text xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="20" id="svg_3" y="160" x="285" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="2,2" stroke-width="0" stroke="#e5e5e5" fill="#e5e5e5">Server</text>
  <text xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="20" id="svg_4" y="160" x="460" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="2,2" stroke-width="0" stroke="#e5e5e5" fill="#e5e5e5">Timer</text>
  <text xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="20" id="svg_5" y="32" x="319" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="2,2" stroke-width="0" stroke="#e5e5e5" fill="#e5e5e5">Application Supervisor</text>
  <text xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="20" id="svg_6" y="412" x="111" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="2,2" stroke-width="0" stroke="#e5e5e5" fill="#e5e5e5">Task</text>
  <text id="svg_9" xml:space="preserve" text-anchor="middle" font-family="Sans-serif" font-size="20" y="411" x="253.01563" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="2,2" stroke-width="0" stroke="#e5e5e5" fill="#e5e5e5">Task</text>
  <circle id="svg_10" r="34.66987" cy="208" cx="584" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="#ffaaaa"/>
  <line id="svg_11" y2="206" x2="548" y1="206" x1="499" stroke-linecap="null" stroke-linejoin="null" stroke-dasharray="null" stroke-width="5" stroke="#e5e5e5" fill="none"/>
 </g>
</svg>

https://github.com/ausimian/timelier

---

### Application Supervisor

```
  def init([]) do
    children = [
      supervisor(Timelier.Task.Supervisor, []),
      worker(Timelier.Server, []),
      worker(Timelier.Timer, [])
    ]

    supervise(children, strategy: :rest_for_one)
  end
```

---

### Task Supervisor

```
  def init([]) do
    children = [
      worker(Timelier.Task.Runner, [], restart: :temporary)
    ]

    supervise(children, strategy: :simple_one_for_one)
  end
```

---

### Design Considerations

>It is pointed out that faults in production software are often ... transient

Jim Gray - [Why Do Computers Stop and What Can Be Done About It?](http://www.hpl.hp.com/techreports/tandem/TR-85.7.pdf) (1985)

---

### Initialization

* Supervision tree initialization is synchronous
  * Each process must return {:ok, state} from init(...)
  * There is no backoff period during restart

* A single failure can prevent system start
  * [It's about the guarantees](http://ferd.ca/it-s-about-the-guarantees.html)
  * [Supervisor backoff pull-request](https://github.com/erlang/otp/pull/1287)
  
* Be careful what you do in init(...)!
  

---

### Error Kernels

* The 'kernel' is the part that _must_ be correct.
  * Identify this part early
  * Make it minimal
  
* Assume the kernel is correct
  * Keep dangerous work outside
  
Notes:
See this in OS Design - kernels should be correct, user programs need not be.

---

### Thanks!

nick@ausimian.net

https://github.com/ausimian
