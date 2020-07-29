### Overview
The flavour of the documentation is primarily educational. It contains papers,
datasheets, style guidelines and team written documentation. The documentation
is organized as follows:
- `adcs`
  - **Attitude Determination and Control System**: How
the satellite can determine its orientation in space and reorient itself if
necessary.
  - *Topics*: Reaction wheels, IMU's, GPS, magnetorquers.
- `comms`
  - **Communication**: How the satellite communicates with the ground station.
  - *Topics*: Transceivers, antenna design and simulation.
- `general`
  - **General team and space related documentation**: Contains things such
as the competition rules and the critical design review.
- `obc`
  - **On Board Computer**: The main computer which organises and executes
tasks on the satellite.
  - *Topics:* Embedded system development, ARM processors, RTOS, STM32.
- `power`
  - **Electronic Power System**: How the satellite stores power from the solar
  panels in batteries and distributes it to the other subsystems.
  - *Topics:* Batteries, solar panels, PCB design.

### Uploading Documentation
We make a distinction between
internal and external documentation.
- internal documentation is generated by team members or organisations and cannot
be easily found on the internet.
- external documentation, such as papers and datasheets, which can be easily found
on the internet.

*Internal documentation* should be formatted in [markdown](https://guides.github.com/features/mastering-markdown/)
if possible. If not it is acceptable to upload the raw file (PDF, PowerPoint, Docx etc.)
to the `files` directory in the associated documentation folder. Please name it sensibly
and don't use spaces or special characters in the title.

For *external documentation* please hyperlink to the document wherever possible.

There should also be a distinction made between project specific documentation
and documentation here. If the documentation you are writing is project specific
related it should go in the docs for the corresponding repo. For example, if
I was writing documentation about a class for the transceiver, that would go
in the docs folder of the repo associated with the developent of that class.
### Help
In the event that links to documentation are broke:
- If the link is to internal documentation in a files directory, it could be
broken because the file was removed, renamed, or the repository path has been
altered. If the document is still contained in the files directory, fix the link
and push the change. If the document has been removed, remove the link.
- If the link is to external documentation, check the [internet archive](https://archive.org/).
If the documentation is on the archive, link to the archive and push the change.
Otherwise remove the link.