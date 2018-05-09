# OPS-PLANT-Report-Script
# Script returning all OPS  fault/jobs where there was a linked job created for PLANT section.

use FaultMan

select OpsJobs.DateFaultReported,OpsJobs.FaultReferenceNo, OpsJobs.Id as JobId,CurrentFieldStaff.UserName as OperatorWhoDispatchePlamber, OpsJobs.CallFaultCode, OpsJobs.CFName, OpsJobs.FFCode as OpsFFCode ,OpsJobs.FFName as OpsFFName ,CurrentFieldStaff.FirstName + ' '+ CurrentFieldStaff.LastName as Plumber, PlantJobs.jobid as PlantJobId, PlantJobs.UserName as OperatorCreatedPlantJob,PlantJobs.FFCode as PlantFFCode,PlantJobs.FFName as PlantFFCodeName,CurrentFieldStaff1.FirstName + ' ' + CurrentFieldStaff1.LastName as PlantFieldWorker, OpsJobs.TownName ,OpsJobs.SuburbName, OpsJobs.LocationName 
from 
(SELECT        dbo.Fault.FaultReferenceNo, dbo.Fault.CreatedTs AS DateFaultReported, dbo.CallFaultCode.CallFaultCode, dbo.CallFaultCode.CFName, dbo.Job.Id, dbo.FieldFaultCode.FFCode, dbo.FieldFaultCode.FFName, dbo.Town.TownName,
                          dbo.Suburb.SuburbName, dbo.Location.LocationName
FROM            dbo.Fault INNER JOIN
                         dbo.Job ON dbo.Fault.FaultReferenceNo = dbo.Job.FaultId INNER JOIN
                         dbo.CallFaultCode ON dbo.Fault.CallfaultCodeId = dbo.CallFaultCode.Id INNER JOIN
                         dbo.FieldFaultCode ON dbo.Job.FFCId = dbo.FieldFaultCode.Id INNER JOIN
                         dbo.Location ON dbo.Job.LocationId = dbo.Location.Id INNER JOIN
                         dbo.Suburb ON dbo.Location.SuburbId = dbo.Suburb.Id INNER JOIN
                         dbo.Town ON dbo.Suburb.TownId = dbo.Town.Id
WHERE        (dbo.Job.SectionId = 69) AND Job.JobStatusId <> 9 and  (dbo.Fault.SectionId = 69)) as OpsJobs 

inner join 

(SELECT        dbo.JobHistory.JobId, dbo.JobHistory.Action, dbo.Employee.FirstName, dbo.Employee.LastName, dbo.JobHistory.CreatedTs, dbo.JobHistory.SectionId,UserName
FROM            dbo.JobHistory INNER JOIN
                         dbo.Employee ON dbo.JobHistory.FieldStaffId = dbo.Employee.Id inner join [user] s on JobHistory.CreatedUserId = s.Id
						 
WHERE        (dbo.JobHistory.Action = 'Current')) as CurrentFieldStaff  on OpsJobs.Id = CurrentFieldStaff.JobId inner join  (select Sectionid,jobid ,max(CreatedTs) as DateCurrent
																					from JobHistory
																					where sectionid = 69 and action = 'Current'
																					group by Sectionid,jobid) 
								as OpsStaffDate on CurrentFieldStaff.JobId = OpsStaffDate.JobId and CurrentFieldStaff.CreatedTs = OpsStaffDate.DateCurrent

Inner join
 
(SELECT        dbo.Job.Id AS jobid, dbo.Job.FaultId, dbo.Job.SectionId, dbo.FieldFaultCode.FFCode, dbo.FieldFaultCode.FFName,UserName 
FROM            dbo.Job INNER JOIN
                         dbo.FieldFaultCode ON dbo.Job.FFCId = dbo.FieldFaultCode.Id inner join [user] s on job.CreatedUserId = s.Id
WHERE        dbo.Job.SectionId = 91 ) as PlantJobs on OpsJobs.FaultReferenceNo = PlantJobs.FaultId

inner join 

(SELECT        dbo.JobHistory.JobId, dbo.JobHistory.Action, dbo.Employee.FirstName, dbo.Employee.LastName, dbo.JobHistory.CreatedTs, dbo.JobHistory.SectionId
FROM            dbo.JobHistory INNER JOIN
                         dbo.Employee ON dbo.JobHistory.FieldStaffId = dbo.Employee.Id
WHERE        (dbo.JobHistory.Action = 'Current')) as CurrentFieldStaff1  on PlantJobs.jobid = CurrentFieldStaff1.JobId inner join  (select Sectionid,jobid ,max(CreatedTs) as DateCurrent
																					from JobHistory
																					where sectionid = 91 and action = 'Current'
																					group by Sectionid,jobid) 
								as PalntStaffDate on CurrentFieldStaff1.JobId = PalntStaffDate.JobId and CurrentFieldStaff1.CreatedTs = PalntStaffDate.DateCurrent


WHERE OpsJobs.DateFaultReported >= '2017-01-01 00:00:00.000' and OpsJobs.DateFaultReported < '2017-09-01 00:00:00.000' and PlantJobs.FFName like '%Backfill%'
