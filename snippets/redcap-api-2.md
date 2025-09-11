Shortcomings of the REDCap API -  
Need for an API 2.0?
==============

## Limitations of the Current REDCap API

Back in the _Motivation_ and _Prerequisits_ sections, I mentioned the export, either via CSV download or API export, of **reports**.

Using reports is powerful, because this leaves full control over the exported data with the project designer/owners. However, currently the REDCap API only supports very coarse rights: **API Import** and **API Export**. Both, the `Export Records` and the `Export Reports` methods are available with API Export right, but they cannot be differentiated further, i.e., it is not possible to grant one without the other.

REDCap does not support field-level access rights. However, with the **combination of report access, data viewing, and data export rights**, in principle, **field-level granularity** could be achieved, _if only it were possible to limit API access to exporting reports_.


But there are more issues with the current implementation of the REDCap API and token management.






## Revised API 2.0


Proposal details

