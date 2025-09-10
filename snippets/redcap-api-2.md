Shortcomings of the REDCap API -  
Need for an API 2.0?
==============

## Limitations of the Current REDCap API

Back in the _Motivation_ section, I mentioned the export, either via CSV download or API export, of **reports**.

This is powerful, because it leaves full control over the exported data with the project designer. However, currently the REDCap API only supports very coarse rigts: **API Import** and **API Export**. Both, the `Export Records` and the `Export Reports` methods are available with API Export right, but they cannot be differentiated further, i.e., it is not possible to grant one without the other.


Explore scenario with report access, data viewing, and data export. Conclude that it is not possible to have basically field level granularity that would in principle be possible with reports, _if_ API access could be limited to just report exports.


Other issues include ...


## Revised API 2.0


Proposal details

