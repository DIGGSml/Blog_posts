# No Translation Needed: How DIGGS Keeps a Data Center Program Moving at Machine Speed

*DIGGS Technical Committee*

---

## The Problem: Data Centers Don't Wait for Paperwork

Hyperscale data center programs move at a pace most geotechnical work was never built for. Dozens of building pads and support structures can be under design at once, foundation decisions gate the construction schedule, and the cost of a stalled decision is measured in weeks of idle steel and concrete, not just billable hours. On these programs, speed and accuracy in moving data from the field to the engineer to the client aren't nice-to-haves. They're the job.

So it's worth asking: on a program with that many moving parts, where does the time actually go? Rarely is it the field work itself. CPT rigs are fast, and a crew can push soundings all day. The time goes into what happens *after* the rig leaves the site: getting the data into a form an engineer can actually use.

## The Old Way: Everyone Speaking a Different Dialect

A single CPT sounding isn't one number. It's raw tip resistance, sleeve friction, and pore pressure (qc, fs, u2) recorded continuously with depth, plus a set of interpreted parameters, soil behavior type index, normalized tip resistance, friction ratio, calculated from that raw data. Add a seismic module and you've also got shear wave velocity. Run a dissipation test and you've got a t50 value that tells you how quickly the soil can drain. All of that belongs together, with the location and provenance that says exactly where and how it was collected.

In the traditional workflow, it usually doesn't stay together. Raw data comes out of the cone's proprietary software as one export. Interpreted parameters get calculated and delivered as a separate spreadsheet, or dropped into a PDF report meant for reading, not for parsing. Dissipation and shear wave results, if they survive the trip at all, tend to end up stranded in side files, disconnected from the sounding they belong to. What should be one record becomes a scattered set of files in formats that don't talk to each other.

Every contributor on a program like this speaks a different dialect: the CPT contractor's field software, the drilling subcontractor's log format, the lab's LIMS export, the engineer's analysis package, the client's GIS database. None of these were built to talk to one another, so someone has to translate. That someone is usually an engineer, re-keying values out of a PDF, reformatting a spreadsheet, or fixing units by hand, hoping nothing gets mistyped along the way. It's slow, and it's exactly the kind of manual step where transcription errors quietly work their way into a foundation design.

## The Turning Point: One Shared Language

DIGGS exists to close that gap. It's an open, vendor-neutral container that holds the full CPT record, raw data, interpreted parameters, metadata, and location, together in a single validated file. No side files, no separate spreadsheet for the parameters your analysis actually needs, no guessing which PDF export is current.

On this data center program, we changed one thing about how field data crossed the line from contractor to engineer: instead of asking our CPT subcontractor, **ConeTec**, for the traditional bundle of PDFs, CSVs, and proprietary exports, we asked for DIGGS.

## What Actually Changed: The Pipeline in Practice

The difference shows up the moment a file lands. ConeTec runs the soundings across the site and delivers each one as a single validated DIGGS file: raw qc/fs/u2, interpreted Ic/SBT/Qtn/Fr, SCPT shear wave velocity, dissipation t50, and full location metadata, all in one container, all traceable back to the same sounding.

Those files feed directly into Schnabel's internal data pipeline. Because the file already speaks DIGGS, the pipeline reads it directly. There's no re-keying values out of a PDF, no reconciling a spreadsheet against a separate log, no reformatting to match what the analysis software expects. Validation happens automatically at the point of ingestion, so a malformed or incomplete file gets caught immediately, not three weeks later when an analyst notices the numbers don't add up. From there, the data flows straight into settlement and liquefaction analysis tools that read DIGGS natively.

The same thing happens on the way out. Deliverables to the client move through the pipeline just as cleanly as they came in, including wrapped DIGGS viewer files the client's engineers can open directly in a browser, no specialized software required, to review a sounding before it ever gets to formal analysis. (We covered how that wrapping works in our [earlier post on the DIGGS Wrapper](../DIGGS_Wrapper/diggs_wrapper_viewer_blog.html).)

Field to engineer to client, without a single manual reformatting step in between.

## See It for Yourself on Geosetta

None of this depends on taking our word for it. **Geosetta.org** is already serving **267,178 CPT soundings** as DIGGS data, a growing share of them as wrapped, browser-ready files exactly like the ones described above.

Head to [this cluster of CPT soundings on the Geosetta web map](https://geosetta.org/web_map/map/#16/46.7683/-92.1197), click a point, and download the sounding as a wrapped DIGGS file. Or skip straight to an example: we've included a real wrapped DIGGS file from a Minnesota DOT CPT sounding right alongside this post: [MnDOT_viewer.html](MnDOT_viewer.html). Download it, double-click it, and raw qc/fs/u2, interpreted parameters, and location data render instantly in your browser, no software and no DIGGS knowledge required.

## Why Speed and Efficiency Mattered Here

On a program with this many pads, this much data, and this little schedule slack, the manual reformatting step was never a minor inconvenience, it was the bottleneck. Every hour spent transcribing a PDF or reconciling a spreadsheet was an hour a foundation decision sat waiting, and every manual re-entry was a chance for a transcription error to work its way into that decision. Removing that step didn't just save time. It removed a category of risk that used to be baked into every data handoff.

At data center scale, that adds up fast. Multiply the time saved and the errors avoided across every sounding on every pad, and a pipeline where data moves without translation stops being a convenience and starts being a competitive advantage.

## The Bigger Lesson

The bottleneck was never really about data volume. It was about translation. Once the field crew, the subcontractor, and the engineering team are all producing and consuming the same format, "data movement" stops being a project risk you have to manage and becomes infrastructure you stop thinking about.

That's not specific to CPT, or to data centers. Any large program with multiple data contributors, drilling subs, CPT crews, labs, instrumentation vendors, the client's own systems, has this same translation problem hiding in it. The fix isn't a faster PDF reader or a better spreadsheet template. It's agreeing, up front, on one language everyone's tools already know how to speak.

## What You Can Do Monday

- **Ask your CPT and drilling subs for DIGGS deliverables**, not PDFs and proprietary exports, on your next program.
- **Use the free [DIGGS Viewer](../DIGGS_Wrapper/diggs_wrapper_viewer_blog.html)** to spot-check incoming files before they leave your desk.
- **Ask your analysis vendor for native DIGGS support** so the data you receive doesn't need a manual detour before it's usable.
- **Archive the project as DIGGS** from the start, so the record you keep is the same one everyone worked from.

## Get Involved

- **GitHub**: [https://github.com/DIGGSml](https://github.com/DIGGSml)
- **Monthly Meetings**: Contact Allen Cadden (acadden@schnabel-eng.com) or Ross Cutts (rcutts@schnabel-eng.com) for meeting invites.
- **Feedback**: Tell us about the data handoffs slowing down your programs.

## Conclusion

Data center work doesn't leave much room for a data pipeline to catch its breath, and on this program, it didn't have to. Once the CPT data crossed the line from contractor to engineer in one shared language instead of a stack of PDFs and side files, the translation step disappeared and took its delays and its errors with it. That's the real payoff of a common standard: not a better-looking file, but one less place for a program to lose time it doesn't have.

---

*Published by ASCE Geo-Institute | DIGGS Technical Committee*
