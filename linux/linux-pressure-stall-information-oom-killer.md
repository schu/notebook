# Pressure Stall Information (PSI) out-of-memory killer

The Linux Kernel's [Pressure Stall Information (PSI)](https://docs.kernel.org/accounting/psi.html)
allows user space services like `systemd-oomd` and `oomd` to take preemptive
steps to avoid OOM in kernel space.

* https://www.freedesktop.org/software/systemd/man/latest/systemd-oomd.service.html

* https://github.com/facebookincubator/oomd
