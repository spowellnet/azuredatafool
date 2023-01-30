

I've been working with data for over 30 years, I remember when databases were new and exciting and 20Gb of disk cost the same sum of money as my car (it was an old banger of a car at the time)

I've been working mostly on the MS stack for 25 years, though have brushed up against various legacy/alterative data systems to recognise most of them by the smell. It took a long time wash of the grime from my MUMPS experience.

I'm based in the UK (North West) and have benefited from the amazing SQL community here in the UK. Local user groups, Data Relay and of course SQL BITS. 

I've been lucky enough to have to coach and train several teams over my last few projects and this is a bunch of that stuff written down. 

There are loads of bloggers/vloggers/tweeters that are way better at most of the things I do than I am but these posts are all a bit off the beaten track, so I thought I'd share.

If you're ever in any doubt about the motivation behind anything I might say, please assume I was probably aiming for funny.






Background

I started a Project in 2019 that was looking at modernising an enterprise data warehouse solution running on an on premise Netezza appliance. The decision had already been made to move it to Azure rather than moving to an alternative cloud provider or to a newer version of the appliance. It had also been decided that the work would be undertaken internally rather than pushing it out to one of the big consultancies.

110Tb and the thick end of 500k tables. Loaded through Informatica and a raft of barely documented business processes using it however they liked. Pretty much everyone in the business doing anything with data had a login to this thing and it had passed through the hands of at least 3 out sourced teams during it's lifetime, about 8 years at that point. 

So they setup a new team at the new head office, the existing team was mostly laid off with a few relocating north and bringing invaluable corporate knowledge with them.

The platform had pretty much been decided as Azure, but few specifics. The eventual technical platform looked like a LOT of Azure Data Factory into a dedicated SQL Pool (Azure DWH as it was at that point) and that's why I was hired as I'd been mauled by the APS and PDW platforms (the predecessors of Azure DWH) in previous consultancy roles.That and my ongoing obsession with ETL frameworks and automation. Why Azure DWH, well mostly because all the business users used SQL to query this thing so a dedicated SQL Pool was the lowest impact on the overall business ecosystem.

The entire team was hired within a few months of each other and into that particular frying pan we all jumped. We had so bitten off more than we could chew :-)



The solution

Without giving any detailed corporate info away the eventual solution was noticeably smaller and noticeably simpler.

After figuring out that huge slices were not really used or were used really badly (5 years worth of tables with names like tableName_YYYY_MM_DD_HH_MM_SS that everyone held onto for dear life) it's about 4,000 end user tables.

About 1200 of these load from source systems every day, mostly old fashioned conventional batch pulls overnight either from readable backups/secondaries or flat files that the legacy systems threw at us at various points during the early hours.

Lots of new stuff driven by flat files, relational sources, the usual corporate cloud services (SalesForce) and some Kafka message based feeds. 

We have 
	an ELT control DB (Azure SQL DB) 
	an Azure Data Factory with lots of parametrised pipelines
	A dedicated SQL Pool
	A collection of LogicApps to take care of various aspects
	A collection of Azure Functions that do the things ADF can't.
	Some DataBricks to undertake various (mostly file based) processing tricks
	
We do
	CI/CD on the ADF
	CI/CD on the DB projects (ELT and DWH)
	CI/CD on the Azure Functions
	CI/CD on the Databricks
	CI/CD on the LogicApps
	Automated build of load processes	

The overnight process loads about 1200 tables ( 1200 STG tables and 1200 ODS tables)
We have another 1200 tables with historic data in that are no longer loading new data
200 ish Dimension/Fact/Snapshot/Aggregate tables
120 ish Extract/Integration tables (providing feeds to other systems)
1000 tables built by different business divisions to provide data within the division and across divisions

A whole backload layer for almost all the tables, temp tables used in various data transformations, backup tables, lots of tooling support and other bells and whistles used throughout the process.




