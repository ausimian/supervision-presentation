# Supervision

#### (or how to successfully embrace failure)

---

### About...

* Worked in a variety of areas
  - Distributed systems, Robotics, Augmented Reality, High-frequency trading

* Long-term Erlang fan
  - Excited to see adoption grow through Elixir

* ausimian
  - Github, Twitter

Notes:
Not a web-guy, not so much experience with Elixir

---

### The Pillars of Robustness

1. Pre-emptive scheduling

2. Process isolation

3. Supervision

Notes:
The BEAM VM takes a unique approach to fault handling.

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

Supervision is a pattern build on primitives.

<svg width="640" height="480" xmlns="http://www.w3.org/2000/svg" xmlns:svg="http://www.w3.org/2000/svg">
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
at Ericsson, and who wrote the first Erlang VM. Termination propagates by default causing all linked proceses to exit. Only one-link per-process pair and not ref-counted. In OTP, rare to use these bifs directly.

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

### You can forget the last 3 slides

(if you writing OTP-compliant applications)

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

