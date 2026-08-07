# Keep Runtime Drivers capability-oriented

Runtime Drivers expose strict common lifecycle semantics plus explicit, versioned capabilities instead of pretending every provider offers one lowest-common-denominator sandbox. Core supplies required and preferred outcomes, while each driver keeps provider mechanisms private and rejects unsupported requirements before side effects; this preserves portability without hiding meaningful differences in isolation, mounts, workload engines, networking, or secure channels.
