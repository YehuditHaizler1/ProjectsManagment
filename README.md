# Finaly project - Task Managment

## Using this technologies:
* MySql
* Web api
* WinForms
* Angular
* php


***
## Web api
### Models
* Worker:
    * Id - int 
    * Name - string - minLength: 2, maxLength:15, reqiered
    * UserName - string - minLength: 2, maxLength:10, reqiered, uniqe
    * Password - string - minLength: 6, maxLength:10, reqiered
    * Email - string  ,minLength: 6, maxLength:30,reqiered
    * job - int - will contain the id of job's type
    * ManagerName - string -minLength: 2, maxLength:15   
* Project:
    * Id - int 
    * Name - string -  minLength: 2, maxLength:25,uniqe, reqiered,
    * CustomerName - string -  minLength: 2, maxLength:15, reqiered,
    * TeamLeader - `Worker` type - reqiered-  will contain the id of the project manager
    * DevelopHoures - int - reqiered
    * QAHoures - int - reqiered
    * UIUXHoures - int - reqiered
    * StartDate - date - reqiered 
    * EndDate - date - reqiered, must be after `StartDate`
* ProjectWorker:
    * Id - int 
    * Worker - `Worker` type
    * Project - `Project` type
    * AllocatedHours - int 
    * WorkHours - int
* Status:
     * Id - int
     * Name - string

* Presence:
     * IdPresentDay - int-Key
     * TimeBegin - DateTime
     * TimeEnd - DateTime 

 * Projects - to reports:
    * ProjectId 
    * ProjectName 
    * CustomerName 
    * DateBegin 
    * DateEnd 
    * DevHours 
    * QAHours 
    * UIUXHours 
    * departments

* Department - to reports:
    * Name 
    * RequiredHours 
    * ActualHours 
    * Workers 
  
    
### Controllers

* Manager controller:
    * Post - add a new project   
    requierd data: 
        * ProjectName
        * CostumerName
        * TeamLeaderName
        * DevelopHoures
        * QAHoures
        * UIUXHoures
        * StartDate 
        * EndDate 
    If the project details is valid - we will add the project to the DB and also add all the workers that bellow to the team-leader to this projects
    * Post - add a new worker
    requierd data: 
          * Name
          * UserName 
          * Password 
          * Email 
          * job 
          * Manager 
    * Get - get all the details that the manager need to the report
    * Get - get all managers - to choose for each worker your manager
    * Get - get all jobs - to choose for each worker your job type
    * Get - get all workers
    * Get - get presence- get the presence for each worker can be filter by month,project and name
    * Put - edit worker's details 
    requierd data: 
          * Name
          * UserName 
          * Email 
          * job 
          * Manager 
    The manager can not change the worker's password     
    * Delete - the manager can delete worker - It possible to delete just a worker and not a team-leader
* TeamLeader controller:
    * Get - get the details and status of the current project that the team-leader manage
    * Get - get the details for each worker that bellow him
    * Get - get all the hours that used in this month 
    * Get - get worker's hours to each worker
    * Put - update worker's hours - The team-leader need to update for each worker
* Worker controller:
    * Post - sign in to the system    
    requierd data: 
        * userName
        * password
    If the user is valid - we will check his status and navigate him to the currect main page, Else - we will return a matching error
    * Post - change password
    requierd data: 
        * userName
        * oldPassword
        * newPassword
    If the user want to change your password
    * Post - auto login
    * Post - the worker can send massage to his team-leader
    * Post - for update the hour that he started to work
    * Get - get the worker details
    * Get - get the project that he work now
    * Get - get all the hours that he worked
***
# Code of each tier

### DAL
```csharp
using System;
using MySql.Data.MySqlClient;
using System.Collections.Generic;

namespace DAL
{
    public class DBAccess
    {
        static MySqlConnection Connection = new MySqlConnection
            (@"SERVER=127.0.0.1;PORT=3307;UID=root;persistsecurityinfo=True;DATABASE=projects_managment;SslMode=none");
 
        public static int? RunNonQuery(string query)
        {
            try
            {
                Connection.Open();
                MySqlCommand command = new MySqlCommand(query, Connection);
                return command.ExecuteNonQuery();
            }
            catch (Exception)
            {
                return null;
            }
            finally
            {
                if (Connection.State != System.Data.ConnectionState.Closed)
                {
                    Connection.Close();
                }
            }

        }

        public static object RunScalar(string query)
        {
            try
            {
                Connection.Open();
                MySqlCommand command = new MySqlCommand(query, Connection);
                return command.ExecuteScalar();
            }
            catch (Exception)
            {
                return null;
            }
            finally
            {
                if (Connection.State != System.Data.ConnectionState.Closed)
                {
                    Connection.Close();
                }
            }
        }

        public static List<T> RunReader<T>(string query, Func<MySqlDataReader, List<T>> func)
        {
            try
            {
                Connection.Open();
                MySqlCommand command = new MySqlCommand(query, Connection);
                MySqlDataReader reader = command.ExecuteReader();
                return func(reader);
            }
            catch (Exception e)
            {
                return null;
            }
            finally
            {
                if (Connection.State != System.Data.ConnectionState.Closed)
                {
                    Connection.Close();
                }
            }
        }
    }
}

```

### BOL
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace BOL
{
    public class Notification
    {
        //primary key
        [Key]
        public int Id { get; set; }

        //required
        [Required]
        public int WorkerId { get; set; }

        //required
        [Required]
        public string Message { get; set; }

        //required
        [Required]
        public int Type { get; set; }
    }
}

```
```csharp
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace BOL
{
    public class Presence
    {
        
        //primary key
        [Key]
        public int Id { get; set; }

        //required
        [Required]
        public int WorkerId { get; set; }

        //required
        //Foreign key from 'Projects' table
        [Required]
        [ForeignKey("Projects")]
        public int ProjectId { get; set; }

        //required
        //DataType is dateTime 
        [Required]
        [DataType(DataType.DateTime)]
        public DateTime BeginningTime { get; set; }

        //required
        //DataType is dateTime 
        [Required]
        [DataType(DataType.DateTime)]
        public DateTime EndTime { get; set; }

    }
}

```
```csharp
using BOL.Validations;
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace BOL
{
    public class Project
    {

        //primary key
        [Key]
        public int Id { get; set; }
        
        //required
        //3 - 20 chars
        [Required]
        [MinLength(3), MaxLength(20)]
        public string Name { get; set; }

        //required
        //3 - 20 chars
        [Required]
        [MinLength(3), MaxLength(20)]
        public string CustomerName { get; set; }

        //required
        //Foreign key from 'Workers' table
        [Required]
        [ForeignKey("Workers")]
        public int ProjectManagerId { get; set; }

        //positive numbers
        [Range(0, int.MaxValue)]
        public int? ProgramHours { get; set; }

        //positive numbers
        [Range(0, int.MaxValue)]
        public int? QAHours { get; set; }

        //positive numbers
        [Range(0, int.MaxValue)]
        public int? UIUXHours { get; set; }

        //required
        //DataType is date 
        [Required]
        [DataType(DataType.Date)]
        public DateTime StartDate { get; set; }

        //required
        //DataType is date
        //end date must be after start date
        [Required]
        [DataType(DataType.Date)]
        [MinEndDate]
        public DateTime EndDate { get; set; }
    }
}

```
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace BOL
{
    public class ProjectWorker
    {
        //primary key
        [Key]
        public int Id { get; set; }

        //required
        //Foreign key from 'Projects' table
        [Required]
        [ForeignKey("Projects")]
        public int ProjectId { get; set; }

        //required
        //Foreign key from 'workers' table
        [Required]
        [ForeignKey("Workers")]
        public int WorkerId { get; set; }

        //required
        //positive numbers -maximum 600
        [Required]
        [Range(0, 600)]
        public int TotalHours { get; set; }
        
    }
}

```
```csharp
using System.ComponentModel.DataAnnotations;

namespace BOL
{
    public class Status
    {
        //primary key
        [Key]
        public int Id { get; set; }

        //required
        //3 - 15 chars
        [Required]
        [MinLength(3),MaxLength(15)]
        public string StatusName { get; set; }
    }
}
```
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using BOL.Validations;

namespace BOL
{
    public class Worker
    {
        //primary key
        [Key]
        public int Id { get; set; }

        //required
        //2 - 10 chars
        [Required]
        [MinLength(2), MaxLength(10)]
        public string UserName { get; set; }

        //required
        //DataType is password
        //64 chars
        [Required]
        [MinLength(64),MaxLength(64)]
        [ValidPassword]
        public string Password { get; set; }

        [MinLength(7),MaxLength(15)]
        public string IpAddress { get; set; }

        //2 - 10 chars
        [MinLength(2), MaxLength(10)]
        public string FirstName { get; set; }

        //required
        //2 - 10 chars
        [Required]
        [MinLength(2), MaxLength(10)]
        public string LastName { get; set; }

        //required
        //DataType is emailAddress
        //pattern of emailAddress
        //5 - 100 chars
        //unique
        [Required]
        [DataType(DataType.EmailAddress)]
        [EmailAddress]
        [MinLength(5), MaxLength(100)]
        //[Unique]
        public string EMail { get; set; }

        //DataType is phoneNumber
        //pattern of phoneNumber
        //9 - 10 chars
        [DataType(DataType.PhoneNumber)]
        [Phone]
        [MinLength(9), MaxLength(10)]
        public string Phone { get; set; }

        //required
        //Foreign key from 'Status' table
        [Required]
        [ForeignKey("Status")]
        public int StatusId { get; set; }

        //Foreign key from 'Worker' table
        [ForeignKey("Workers")]
        public int? ManagerId { get; set; }

        //positive numbers
        [Range(0, 600)]
        public int? TotalHours { get; set; }
    }
}

```


### BLL
```csharp
using BOL;
using BOL.Help;
using DAL;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;

namespace BLL
{
    public class LogicUser
    {

        public static Worker AutoLogIn(string ipAddress)
        {
            string query = $"SELECT id,userName,ipAddress,firstName,lastName,eMail,phone,statusId,managerId,totalHours " +
                $"FROM projects_managment.workers " +
                $"WHERE ipAddress = '{ipAddress}' ";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    workers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        IpAddress = reader.GetString(2),
                        FirstName = reader.GetString(3),
                        LastName = reader.GetString(4),
                        EMail = reader.GetString(5),
                        Phone = reader.GetString(6),
                        StatusId = reader.GetInt32(7),
                        ManagerId = reader[8] as int?,
                        TotalHours = reader[9] as int?
                    });
                }
                return workers;
            };

            List<Worker> worker = DBAccess.RunReader(query, func);
            if (worker != null && worker.Count > 0)
                return worker[0];
            return null;
        }

        //LogOut - only when the details are matching
        public static bool LogOut(Worker worker)
        {
            string query = $"UPDATE projects_managment.workers SET ipAddress = NULL WHERE id = {worker.Id} AND ipAddress = '{worker.IpAddress}' ";

            return DBAccess.RunNonQuery(query) == 1;
        }

        //Login - Get email and password, Check if the user exists. If exists - return the user, Else return null.
        public static Worker LogIn(string eMail, string password)
        {
            string query = $"SELECT id,userName,ipAddress,firstName,lastName,eMail,phone,statusId,managerId,totalHours " +
                $"FROM projects_managment.workers " +
                $"WHERE eMail = '{eMail}' AND password = '{password}' ";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    Worker w = new Worker();
                    w.Id = reader.GetInt32(0);
                    w.UserName = reader.GetString(1);
                    w.IpAddress = (reader[2] is DBNull) ? null : reader.GetString(2);
                    w.FirstName = reader.GetString(3);
                    w.LastName = reader.GetString(4);
                    w.EMail = reader.GetString(5);
                    w.Phone = reader.GetString(6);
                    w.StatusId = reader.GetInt32(7);
                    w.ManagerId = reader[8] as int?;
                    w.TotalHours = reader[9] as int?;

                    workers.Add(w);
                }
                return workers;
            };

            List<Worker> worker = DBAccess.RunReader(query, func);
            if (worker != null && worker.Count > 0)
                return worker[0];
            return null;
        }

        //RememberMe - Get workerId and ipAddress, 
        //Return if succeeded updating the match worker's ipAddress to the new address
        //(can't update when the worker add another ipAddress)
        public static bool RememberMe(int workerId, string ipAddress)
        {
            string query = $"UPDATE projects_managment.workers SET ipAddress = '{ipAddress}' WHERE id = {workerId} AND ipAddress IS NULL";

            return DBAccess.RunNonQuery(query) == 1;
        }

        public static List<Notification> GetNotifications(int workerId)
        {
            string query = $"SELECT * FROM projects_managment.notifications WHERE workerId = {workerId}";

            Func<MySqlDataReader, List<Notification>> func = (reader) =>
            {
                List<Notification> notifications = new List<Notification>();
                while (reader.Read())
                {
                    notifications.Add(new Notification
                    {
                        Id = reader.GetInt32(0),
                        WorkerId = reader.GetInt32(1),
                        Message = reader.GetString(2),
                        Type = reader.GetInt32(3)
                    });
                }
                return notifications;
            };

            List<Notification> notificationsList = DBAccess.RunReader(query, func);
            return notificationsList;
        }

        public static bool DeleteNotification(int notificationId)
        {
            string query = $"DELETE FROM projects_managment.notifications WHERE id = {notificationId}";

            return DBAccess.RunNonQuery(query) == 1;
        }
        public static List<Project> GetProjectsToWorker(int workerId)
        {
            string query = $"SELECT p.* FROM projects_managment.projects p " +
                $"JOIN projects_managment.projectsworker pw ON p.id = pw.projectId " +
                $"JOIN workers w ON pw.workerId = w.id WHERE w.id = {workerId} AND p.startDate < NOW() AND p.endDate > NOW()";

            Func<MySqlDataReader, List<Project>> func = (reader) =>
            {
                List<Project> projects = new List<Project>();
                while (reader.Read())
                {
                    projects.Add(new Project
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        CustomerName = reader.GetString(2),
                        ProjectManagerId = reader.GetInt32(3),
                        ProgramHours = reader[4] as int?,
                        QAHours = reader[5] as int?,
                        UIUXHours = reader[6] as int?,
                        StartDate = reader.GetDateTime(7),
                        EndDate = reader.GetDateTime(8)
                    });
                }
                return projects;
            };

            List<Project> workerProjectsList = DBAccess.RunReader(query, func);
            return workerProjectsList;
        }

        public static bool IsPassiveWorker(int workerId)
        {
            string query = $"SELECT COUNT(*) FROM projects_managment.presences WHERE workerId = {workerId} AND endTime IS NULL ";

            return (long)DBAccess.RunScalar(query) == 0;
        }

        public static bool AddPresence(Presence presence)
        {
            string query = $"INSERT INTO projects_managment.presences (workerId, projectId, beginningTime, endTime) VALUES ({presence.WorkerId},{presence.ProjectId},'{presence.BeginningTime.ToString("yyyy-MM-dd HH:mm:ss")}', NULL);";

            return DBAccess.RunNonQuery(query) == 1;
        }

        public static int GetCurrentPresenceId(Presence presence)
        {
            string query = $"SELECT id FROM projects_managment.presences WHERE " +
                $"workerId={presence.WorkerId} AND projectId={presence.ProjectId} AND beginningTime='{presence.BeginningTime.ToString("yyyy-MM-dd HH:mm:ss")}' ;";

            return (Int32)DBAccess.RunScalar(query);
        }

        //UpdateEndTime - Get presenceId and endtime, Update in D.B. the endTime in the correct object (by the presenceId)
        public static bool UpdateEndTime(int presenceId, DateTime endTime)
        {
            string query = $"UPDATE projects_managment.presences SET endTime = '{endTime.ToString("yyyy-MM-dd HH:mm:ss")}' WHERE id = {presenceId}; ";

            return DBAccess.RunNonQuery(query) == 1;
        }

        //GetHoursStatusToWorker - Get workerId, Return his hoursStatus
        public static List<HoursStatusW> GetHoursStatusToWorker(int workerId)
        {
            string query = $"SELECT p.*, " +
                $"pw.totalhours * 60 AS 'Required Minutes', " +
                $"(SELECT SUM( TIMESTAMPDIFF(MINUTE, pr.beginningTime, pr.endTime)) " +
                $"FROM projects_managment.presences pr WHERE pr.workerId=pw.workerId AND pr.projectId=pw.projectId) AS 'Actual Minutes' " +
                $"FROM projects_managment.projects p JOIN projects_managment.projectsworker pw " +
                $"ON p.id=pw.projectId " +
                $"WHERE pw.workerId = {workerId}";
            //string query = $"SELECT pw.workerId, pw.projectId, pw.totalhours * 60 AS 'Required Hours', " +
            //    $"(SELECT SUM( TIMESTAMPDIFF(MINUTE, pr.beginningTime, pr.endTime)) FROM projects_managment.presences pr " +
            //    $"WHERE pr.workerId=pw.workerId AND pr.projectId=pw.projectId) AS 'Actual Hours' " +
            //    $"FROM projects_managment.projectsworker pw WHERE pw.workerId = {workerId} ";

            Func<MySqlDataReader, List<HoursStatusW>> func = (reader) =>
            {
                List<HoursStatusW> hoursStatus = new List<HoursStatusW>();
                HoursStatusW hoursStatusW;
                while (reader.Read())
                {
                    hoursStatusW = new HoursStatusW
                    {
                        Project = new Project
                        {
                            Id = reader.GetInt32(0),
                            Name = reader.GetString(1),
                            CustomerName = reader.GetString(2),
                            ProjectManagerId = reader.GetInt32(3),
                            ProgramHours = reader[4] as int?,
                            QAHours = reader[5] as int?,
                            UIUXHours = reader[6] as int?,
                            StartDate = reader.GetDateTime(7),
                            EndDate = reader.GetDateTime(8)
                        },
                        RequiredHours = (reader.GetFloat(9) as float?) / 60,
                    };
                    if (reader[10] is DBNull)
                        hoursStatusW.ActualHours = null;
                    else
                        hoursStatusW.ActualHours = (reader.GetFloat(10) as float?) / 60;
                    hoursStatus.Add(hoursStatusW);
                }
                return hoursStatus;
            };
            return DBAccess.RunReader(query, func);
        }

    }
}
```
```csharp
using BOL;
using BOL.Help;
using DAL;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;

namespace BLL
{
    public class LogicTeamLeader
    {

        //GetPresencesToTeamLeader - Get team-leader-id, Return the presences of the workers are working in that team-leader's staff 
        //public static List<Presence> GetPresencesToTeamLeader(int projectManagerId)
  
        //GetProjectsToTeamLeader - Get team-leader-id, Return the projects he manages
        public static List<Project> GetProjectsToTeamLeader(int teamLeaderId)
        {
            string query = $"SELECT * FROM projects_managment.projects WHERE projectManagerId = {teamLeaderId} ";

            Func<MySqlDataReader, List<Project>> func = (reader) =>
            {
                List<Project> projects = new List<Project>();
                while (reader.Read())
                {
                    projects.Add(new Project
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        CustomerName = reader.GetString(2),
                        ProjectManagerId = reader.GetInt32(3),
                        ProgramHours = reader[4] as int?,
                        QAHours = reader[5] as int?,
                        UIUXHours = reader[6] as int?,
                        StartDate = reader.GetDateTime(7),
                        EndDate = reader.GetDateTime(8)
                    });
                }
                return projects;
            };

            return DBAccess.RunReader(query, func);
        }

        //GetWorkersToTeamLeader - Get team-leader-id, Return the workers he manages
        public static List<Worker> GetWorkersToTeamLeader(int teamLeaderId)
        {
            string query = $"SELECT id,userName,firstName,lastName,eMail,phone,statusId,managerId,totalHours FROM projects_managment.workers WHERE managerId = {teamLeaderId}";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    workers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5),
                        StatusId = reader.GetInt32(6),
                        ManagerId = reader[7] as int?,
                        TotalHours = reader[8] as int?
                    });
                }
                return workers;
            };

            return DBAccess.RunReader(query, func);
        }

        //UpdateTotalHoursToWorker - ??????????????
        public static bool UpdateTotalHoursToWorker(ProjectWorker projectWorker)
        {
            string query = $"UPDATE projects_managment.projectsworker SET totalHours = {projectWorker.TotalHours} " +
                $"WHERE projectId = {projectWorker.ProjectId} AND workerId = {projectWorker.WorkerId} ";

            return DBAccess.RunNonQuery(query) == 1;
        }

        //GetAllStatus - Return the statusses
        public static List<Status> GetAllStatus()
        {
            string query = $"SELECT * FROM projects_managment.status";

            Func<MySqlDataReader, List<Status>> func = (reader) =>
            {
                List<Status> statusList = new List<Status>();
                while (reader.Read())
                {
                    statusList.Add(new Status
                    {
                        Id = reader.GetInt32(0),
                        StatusName = reader.GetString(1)
                    });
                }
                return statusList;
            };

            return DBAccess.RunReader(query, func);
        }

        //GetHoursStatusToProject - Get projectId, Return its hoursStatus
        public static List<HoursStatusTL> GetHoursStatusToProject(int projectId)
        {
            string query = $"SELECT w.id, w.userName, w.firstName, w.lastName, w.eMail, w.phone, w.statusId, w.managerId, w.totalHours, " +
                $"pw.totalhours * 60 AS 'Required Minutes', " +
                $"(SELECT SUM( TIMESTAMPDIFF(MINUTE, pr.beginningTime, pr.endTime)) FROM projects_managment.presences pr " +
                $"WHERE pr.workerId=pw.workerId AND pr.projectId=pw.projectId) AS 'Actual Minutes' " +
                $"FROM projects_managment.workers w JOIN projects_managment.projectsworker pw " +
                $"ON w.id=pw.workerId " +
                $"WHERE pw.projectId = {projectId} ";

            Func<MySqlDataReader, List<HoursStatusTL>> func = (reader) =>
            {
                List<HoursStatusTL> hoursStatus = new List<HoursStatusTL>();
                HoursStatusTL hoursStatusTL;

                while (reader.Read())
                {
                    hoursStatusTL = new HoursStatusTL
                    {
                        Worker = new Worker
                        {
                            Id = reader.GetInt32(0),
                            UserName = reader.GetString(1),
                            FirstName = reader.GetString(2),
                            LastName = reader.GetString(3),
                            EMail = reader.GetString(4),
                            Phone = reader.GetString(5),
                            StatusId = reader.GetInt32(6),
                            ManagerId = reader[7] as int?,
                            TotalHours = reader[8] as int?
                        },
                        RequiredHours = (reader[9] is DBNull) ? 0 : (reader[9] as long?) / 60,
                        ActualHours = (reader[10] is DBNull) ? 0 : (float)(reader[10] as decimal?) / 60
                    };
                    
                    hoursStatus.Add(hoursStatusTL);
                }
                return hoursStatus;
            };
            return DBAccess.RunReader(query, func);
        }

        //GetHoursStatusToWorker
        public static List<HoursStatusW> GetHoursStatusToWorker(int workerId, int teamLeaderId)
        {
            string query = $"SELECT p.*, pw.totalhours * 60 AS 'Required Minutes', " +
                $"(SELECT SUM(TIMESTAMPDIFF(MINUTE, pr.beginningTime, pr.endTime)) FROM projects_managment.presences pr " +
                $"WHERE pr.workerId = pw.workerId AND pr.projectId = pw.projectId) AS 'Actual Minutes' " +
                $"FROM projects_managment.projects p JOIN projects_managment.projectsworker pw " +
                $"ON p.id = pw.projectId " +
                $"WHERE pw.workerId = (SELECT id " +
                $"FROM projects_managment.workers " +
                $"WHERE id = {workerId} AND managerId = {teamLeaderId}) ";
            //string query = $"SELECT * FROM projects_managment.projects WHERE id IN " +
            //    $"(SELECT projectId FROM projects_managment.projectsworker WHERE workerId = " +
            //    $"(SELECT id FROM projects_managment.workers WHERE id ={workerId} AND managerId ={teamLeaderId}))";

            Func<MySqlDataReader, List<HoursStatusW>> func = (reader) =>
            {
                List<HoursStatusW> hoursStatus = new List<HoursStatusW>();
                HoursStatusW hoursStatusW;
                while (reader.Read())
                {
                    hoursStatusW = new HoursStatusW
                    {
                        Project = new Project
                        {
                            Id = reader.GetInt32(0),
                            Name = reader.GetString(1),
                            CustomerName = reader.GetString(2),
                            ProjectManagerId = reader.GetInt32(3),
                            ProgramHours = reader[4] as int?,
                            QAHours = reader[5] as int?,
                            UIUXHours = reader[6] as int?,
                            StartDate = reader.GetDateTime(7),
                            EndDate = reader.GetDateTime(8)
                        },
                        RequiredHours = (reader[9] is DBNull) ? 0 : (reader[9] as long?) / 60,
                        ActualHours = (reader[10] is DBNull) ? 0 : (float)(reader[10] as decimal?) / 60
                    };
                    hoursStatus.Add(hoursStatusW);
                }
                return hoursStatus;
            };

            return DBAccess.RunReader(query, func);
        }

    }
}

```
```csharp
using BOL;
using BOL.Help;
using DAL;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;

namespace BLL
{
    public static class LogicManager
    {
        //GetAllStatus - Return the statusses
        public static List<Status> GetAllStatus()
        {
            string query = $"SELECT * FROM projects_managment.status";

            Func<MySqlDataReader, List<Status>> func = (reader) =>
            {
                List<Status> statusList = new List<Status>();
                while (reader.Read())
                {
                    statusList.Add(new Status
                    {
                        Id = reader.GetInt32(0),
                        StatusName = reader.GetString(1)
                    });
                }
                return statusList;
            };

            return DBAccess.RunReader(query, func);
        }

        //GetAllCompanyWorkers - Return the company-workers
        public static List<Worker> GetAllCompanyWorkers()
        {
            string query = $"SELECT id,userName,firstName,lastName,eMail,phone,statusId,managerId,totalHours " +
                $"FROM projects_managment.workers ";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    workers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5),
                        StatusId = reader.GetInt32(6),
                        ManagerId = reader[7] as int?,
                        TotalHours = reader[8] as int?
                    });
                }
                return workers;
            };

            return DBAccess.RunReader(query, func);
        }

        //GetAllWorkers - Return all workers (program, QA, UI/UX)
        public static List<Worker> GetAllWorkers()
        {
            string query = $"SELECT w.id,w.userName,w.firstName,w.lastName,w.eMail,w.phone,w.statusId,w.managerId,w.totalHours " +
                $"FROM projects_managment.workers w JOIN projects_managment.status s ON w.statusId = s.id " +
                $"WHERE s.statusName in ('Programmer', 'QA', 'UI/UX') ; ";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    workers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5),
                        StatusId = reader.GetInt32(6),
                        ManagerId = reader[7] as int?,
                        TotalHours = reader[8] as int?
                    });
                }
                return workers;
            };

            return DBAccess.RunReader(query, func);
        }

        //GetAllTeamLeaders - Return all team-leaders
        public static List<Worker> GetAllTeamLeaders()
        {
            string query = $"SELECT w.id,w.userName,w.firstName,w.lastName,w.eMail,w.phone,w.statusId,w.managerId,w.totalHours " +
                $"FROM projects_managment.workers w JOIN projects_managment.status s ON w.statusId = s.id " +
                $" WHERE s.statusName = 'ProjectManager'";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> ProjectManager = new List<Worker>();
                while (reader.Read())
                {
                    ProjectManager.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5),
                        StatusId = reader.GetInt32(6),
                        ManagerId = reader[7] as int?,
                        TotalHours = reader[8] as int?
                    });
                }
                return ProjectManager;
            };

            return DBAccess.RunReader(query, func);
        }

        //GetAllProjects - Return the projects
        public static List<Project> GetAllProjects()
        {
            string query = $"SELECT * FROM projects_managment.projects";

            Func<MySqlDataReader, List<Project>> func = (reader) =>
            {
                List<Project> projectiLst = new List<Project>();
                while (reader.Read())
                {
                    projectiLst.Add(new Project
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        CustomerName = reader.GetString(2),
                        ProjectManagerId = reader.GetInt32(3),
                        ProgramHours = reader[4] as int?,
                        QAHours = reader[5] as int?,
                        UIUXHours = reader[6] as int?,
                        StartDate = reader.GetDateTime(7),
                        EndDate = reader.GetDateTime(8)
                    });
                }
                return projectiLst;
            };

            return DBAccess.RunReader(query, func);
        }

        //AddProject
        public static bool AddProject(Project project)
        {

            // INSERT INTO projects_managment.projects(name, customerName, projectManagerId, programHours, QAHours, UIUXHours, startDate, endDate)
            //VALUES('MemoryGame', 'Seldat', 2, 30, 15, 10, '2018-09-09', '2018-09-29');
            string query = $"INSERT INTO projects_managment.projects(name, customerName, projectManagerId, programHours, QAHours, UIUXHours, startDate, endDate)" +
                $" VALUES ( '{project.Name}','{project.CustomerName}',{project.ProjectManagerId},{project.ProgramHours},{project.QAHours},{project.UIUXHours},'{project.StartDate.ToString("yyyy-MM-dd")}','{project.EndDate.ToString("yyyy-MM-dd")}' ) ";
            return DBAccess.RunNonQuery(query) == 1;
        }

        //GetCurrentProjectId - Return projectId by its name
        public static int GetCurrentProjectId(string projectName)
        {
            string query = $"SELECT id FROM projects_managment.projects WHERE name = '{projectName}'";

            return (Int32)DBAccess.RunScalar(query);
        }

        //GetWorkersToManagers - Get team-leader-id, Return the list of his staff
        public static List<Worker> WorkersToCurrentManager(int managerId)
        {
            string query = $"SELECT id,userName,firstName,lastName,eMail,phone,statusId,managerId,totalHours " +
                $"FROM projects_managment.workers WHERE managerId = { managerId }";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workersToManager = new List<Worker>();
                while (reader.Read())
                {
                    workersToManager.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5),
                        StatusId = reader.GetInt32(6),
                        ManagerId = reader[7] as int?,
                        TotalHours = reader[8] as int?
                    });
                }
                return workersToManager;
            };

            return DBAccess.RunReader(query, func);
        }

        //GetNotAllowedWorkersToProject - Get projectId, Return the workers are not allowed to work on that project
        public static List<Worker> GetNotAllowedWorkersToProject(int projectId)
        {
            string query = $"SELECT id,userName,firstName,lastName,eMail,phone,statusId,managerId,totalHours " +
                $"FROM projects_managment.workers WHERE id NOT IN " +
                $"( SELECT workerId FROM projects_managment.projectsworker WHERE projectId = {projectId}) AND statusId IN (3,4,5)";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> notAllowedWorkers = new List<Worker>();
                while (reader.Read())
                {
                    notAllowedWorkers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5),
                        StatusId = reader.GetInt32(6),
                        ManagerId = reader[7] as int?,
                        TotalHours = reader[8] as int?
                    });
                }
                return notAllowedWorkers;
            };

            return DBAccess.RunReader(query, func);
        }

        //AddWorkerToProject - Get projectId and workerId, Add the correct worker to the correct project's staff
        public static bool AddWorkerToProject(int projectId, int workerId,int totalHours)
        {
            string query = $" INSERT INTO projects_managment.projectsworker (projectId,totalHours,workerId) VALUES ({projectId},{totalHours},{workerId})";
            return DBAccess.RunNonQuery(query) == 1;
        }

        //AddWorker
        public static bool AddWorker(Worker worker)
        {
            string query = $"INSERT INTO projects_managment.workers (userName, password, firstName, lastName, eMail, phone, statusId, managerId, totalHours)" +
                $" VALUES ('{worker.UserName}','{worker.Password}','{worker.FirstName}','{worker.LastName}','{worker.EMail}','{worker.Phone}',{worker.StatusId},";
            if (worker.ManagerId == null)
                query += $"null,null)";
            else query += $"{worker.ManagerId},{worker.TotalHours})";

            return DBAccess.RunNonQuery(query) == 1;
        }

        //GetWorkerById
        public static Worker GetWorkerById(int workerId)
        {
            string query = $"SELECT id,userName,firstName,lastName,eMail,phone,statusId,managerId,totalHours " +
                $"FROM projects_managment.workers WHERE Id = { workerId} ";
            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    workers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5),
                        StatusId = reader.GetInt32(6),
                        ManagerId = reader[7] as int?,
                        TotalHours = reader[8] as int?
                    });
                }
                return workers;
            };

            List<Worker> worker = DBAccess.RunReader(query, func);
            if (worker != null && worker.Count > 0)
                return worker[0];
            return null;
        }

        //UpdateWorker - Update all properties, without the password
        public static bool UpdateWorker(Worker worker)
        {
            string query = $"UPDATE projects_managment.workers SET " +
                $"userName = '{worker.UserName}', firstName = '{worker.FirstName}', lastName = '{worker.LastName}', eMail = '{worker.EMail}', phone = '{worker.Phone}', " +
                $"statusId = {worker.StatusId}, managerId = {worker.ManagerId}, totalHours = {worker.TotalHours} WHERE id = { worker.Id }";
            return DBAccess.RunNonQuery(query) == 1;
        }

        //RemoveWorker
        public static bool RemoveWorker(int workerId)
        {
            string query;
            try
            {
                query = $" DELETE FROM projects_managment.projectsworker WHERE workerId = {workerId}";
                DBAccess.RunNonQuery(query);

                query = $" DELETE FROM projects_managment.presences WHERE workerId = {workerId}";
                DBAccess.RunNonQuery(query);


                query = $"DELETE FROM projects_managment.workers WHERE Id = {workerId}";
                DBAccess.RunNonQuery(query);

            }
            catch (Exception e)
            {
                return false;
            }
            return true;
        }

        //UpdateTeamLeaderToWorker - Update teamLeaderId to the correct worker
        public static bool UpdateTeamLeaderToWorker(int workerId, int teamLeaderId)
        {
            string query = $"UPDATE projects_managment.workers SET managerId= {teamLeaderId} WHERE id= {workerId}";
            return DBAccess.RunNonQuery(query) == 1;
        }

        //Notification
        public static string GetCurrentProjectName(int projectId)
        {
            string query = $"SELECT name FROM projects_managment.projects WHERE id = '{projectId}'";

            return (string)DBAccess.RunScalar(query);
        }

        public static string GetTeamLeaderName(int teamLeaderId)
        {
            string query = $"SELECT userName FROM projects_managment.workers WHERE id = '{teamLeaderId}'";

            return (string)DBAccess.RunScalar(query);
        }

        public static bool AddNotification(Notification notification)
        {
            string query = $"INSERT INTO  projects_managment.notifications (workerId,message,type) " +
                $"VALUES({notification.WorkerId},'{notification.Message}', {notification.Type})";

            return DBAccess.RunNonQuery(query) == 1;
        }
        //------------Trees------------------

        //GetTreeProjects
        public static List<_1_1_Project> GetTreeProjects()
        {
            string query;
            List<_1_1_Project> projectsList;

            //1 - get all projects
            query = $"SELECT p.id,p.name,p.customerName, " +
                $"(SELECT userName FROM projects_managment.workers WHERE id=p.projectManagerId) AS 'TeamLeader' " +
                $",p.programHours AS 'RequiredProgramHours',p.QAHours AS 'RequiredQAHours',p.UIUXHours AS 'RequiredUIUXHours', p.startDate,p.endDate " +
                $"FROM projects_managment.projects p " +
                $"WHERE p.startDate <= DATE(NOW()) AND p.endDate >= DATE(NOW())";

            Func<MySqlDataReader, List<_1_1_Project>> funcProjects = (reader) =>
            {
                List<_1_1_Project> projects = new List<_1_1_Project>();
                while (reader.Read())
                {
                    projects.Add(new _1_1_Project
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        CustomerName = reader.GetString(2),
                        TeamLeader = reader.GetString(3),
                        RequiredProgramHours = reader[4] as int?,
                        RequiredQAHours = reader[5] as int?,
                        RequiredUIUXHours = reader[6] as int?,
                        StartDate = reader.GetDateTime(7),
                        EndDate = reader.GetDateTime(8),
                        Departments = new List<_1_2_Department>()
                    });
                }
                return projects;
            };
            projectsList = DBAccess.RunReader(query, funcProjects);


            //2 - get workers to projects
            Func<MySqlDataReader, List<_1_3_Worker>> funcWorkers;
            List<_1_3_Worker> workersList;

            foreach (_1_1_Project project in projectsList)
            {
                query = $"SELECT w.id,w.userName,w.firstName,w.lastName,w.eMail,w.phone,w.statusId, " +
                    $"(SELECT userName FROM projects_managment.workers WHERE id=w.managerId) AS 'TeamLeader', " +
                    $"(SELECT pw.totalHours FROM projects_managment.projectsworker pw WHERE pw.workerId=w.id AND pw.projectId = {project.Id}) * 60 AS 'Required Minutes', " +
                    $"(SELECT SUM( TIMESTAMPDIFF(MINUTE, pr.beginningTime, pr.endTime)) FROM projects_managment.presences pr WHERE pr.workerId=w.id AND pr.projectId = {project.Id}) AS 'Actual Minutes' " +
                    $"FROM projects_managment.workers w WHERE w.statusId IN (3,4,5) " +
                    $"AND w.id IN (SELECT workerId FROM projects_managment.projectsworker pw WHERE pw.projectId = {project.Id}) ";
                funcWorkers = (reader) =>
                {
                    List<_1_3_Worker> workers = new List<_1_3_Worker>();

                    while (reader.Read())
                    {
                        workers.Add(new _1_3_Worker
                        {
                            Id = reader.GetInt32(0),
                            UserName = reader.GetString(1),
                            FirstName = reader.GetString(2),
                            LastName = reader.GetString(3),
                            EMail = reader.GetString(4),
                            Phone = reader.GetString(5),
                            StatusId = reader.GetInt32(6),
                            TeamLeader = reader.GetString(7),
                            RequiredHours = (reader[8] is DBNull) ? 0 : (reader[8] as long?) / 60,
                            ActualHours = (reader[9] is DBNull) ? 0 : (float)(reader[9] as decimal?) / 60
                        });
                    }

                    return workers;
                };

                workersList = DBAccess.RunReader(query, funcWorkers);


                //3 - get presences to worker
                Func<MySqlDataReader, List<_0_Presence>> funcPresences;

                foreach (_1_3_Worker worker in workersList)
                {
                    query = $"SELECT pr.beginningTime, pr.endTime FROM projects_managment.presences pr WHERE pr.workerId = {worker.Id} AND pr.projectId = {project.Id} AND endTime IS NOT NULL";

                    funcPresences = (reader) =>
                    {
                        List<_0_Presence> presences = new List<_0_Presence>();
                        while (reader.Read())
                        {
                            presences.Add(new _0_Presence
                            {
                                BeginningTime = reader.GetDateTime(0),
                                EndTime = reader.GetDateTime(1)
                            });
                        }
                        return presences;
                    };
                    worker.Presences = DBAccess.RunReader(query, funcPresences);


                    //4 - push the worker to the match department
                    _1_2_Department matchDepartment;
                    switch (worker.StatusId)
                    {
                        case 3:
                            {
                                matchDepartment = project.Departments.Find(d => d.Name == "Program");
                                if (matchDepartment == null)
                                {
                                    //first time
                                    project.Departments.Add(new _1_2_Department
                                    {
                                        Name = "Program",
                                        Workers = new List<_1_3_Worker>(),
                                        RequiredHours = project.RequiredProgramHours,
                                        ActualHours = 0
                                    });
                                    matchDepartment = project.Departments.Find(d => d.Name == "Program");
                                }
                                matchDepartment.Workers.Add(worker);
                                matchDepartment.ActualHours += worker.ActualHours;
                                project.ActualProgramHours = matchDepartment.ActualHours;
                            }
                            break;
                        case 4:
                            {
                                matchDepartment = project.Departments.Find(d => d.Name == "QA");
                                if (matchDepartment == null)
                                {
                                    //first time
                                    project.Departments.Add(new _1_2_Department
                                    {
                                        Name = "QA",
                                        Workers = new List<_1_3_Worker>(),
                                        RequiredHours = project.RequiredQAHours,
                                        ActualHours = 0
                                    });
                                    matchDepartment = project.Departments.Find(d => d.Name == "QA");
                                }
                                matchDepartment.Workers.Add(worker);
                                matchDepartment.ActualHours += worker.ActualHours;
                                project.ActualQAHours = matchDepartment.ActualHours;
                            }
                            break;
                        case 5:
                            {
                                matchDepartment = project.Departments.Find(d => d.Name == "UI/UX");
                                if (matchDepartment == null)
                                {
                                    //first time
                                    project.Departments.Add(new _1_2_Department
                                    {
                                        Name = "UI/UX",
                                        Workers = new List<_1_3_Worker>(),
                                        RequiredHours = project.RequiredUIUXHours,
                                        ActualHours = 0
                                    });
                                    matchDepartment = project.Departments.Find(d => d.Name == "UI/UX");
                                }
                                matchDepartment.Workers.Add(worker);
                                matchDepartment.ActualHours += worker.ActualHours;
                                project.ActualUIUXHours = matchDepartment.ActualHours;
                            }
                            break;
                        default:
                            break;
                    }
                }
            }

            return projectsList;
        }

        //GetTreeWorkers
        public static List<_2_1_Worker> GetTreeWorkers()
        {
            string query;
            List<_2_1_Worker> workersList;

            //1- get all workers
            query = $"SELECT w.id, w.userName, w.firstName, w.lastName, w.eMail, w.phone, " +
                $"(SELECT statusName FROM projects_managment.status WHERE id=w.statusId) AS 'status', " +
                $"(SELECT userName FROM projects_managment.workers WHERE id=w.managerId) AS 'Team leader', w.totalHours  " +
                $"FROM projects_managment.workers w WHERE w.statusId IN (3,4,5) ";

            Func<MySqlDataReader, List<_2_1_Worker>> funcWorkers = (reader) =>
            {
                List<_2_1_Worker> workers = new List<_2_1_Worker>();
                while (reader.Read())
                {
                    workers.Add(new _2_1_Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5),
                        Status = reader.GetString(6),
                        TeamLeader = reader.GetString(7),
                        RequiredHours = reader[8] as int?,
                        ActualHours = 0,
                        Projects = new List<_2_2_Project>()
                    });
                }
                return workers;
            };

            workersList = DBAccess.RunReader(query, funcWorkers);


            //2- get projects to workers
            Func<MySqlDataReader, List<_2_2_Project>> funcProjects;
            List<_2_2_Project> projectsList;

            foreach (_2_1_Worker worker in workersList)
            {
                query = $"SELECT p.id, p.name ,pw.totalHours * 60 AS 'Required Minutes', " +
                    $"(SELECT SUM( TIMESTAMPDIFF(MINUTE, pr.beginningTime, pr.endTime)) " +
                    $"FROM projects_managment.presences pr WHERE pr.workerId = {worker.Id} AND pr.projectId=p.id) AS 'Actual Minutes' " +
                    $"FROM projects_managment.projects p JOIN projects_managment.projectsworker pw ON p.id = pw.projectId " +
                    $"WHERE pw.workerId = {worker.Id} AND p.startDate <= DATE(NOW()) AND p.endDate >= DATE(NOW()) ";

                funcProjects = (reader) =>
                {
                    List<_2_2_Project> projects = new List<_2_2_Project>();
                    while (reader.Read())
                    {
                        projects.Add(new _2_2_Project
                        {
                            Id = reader.GetInt32(0),
                            Name = reader.GetString(1),
                            RequiredHours = (reader[2] is DBNull) ? 0 : (reader[2] as long?) / 60,
                            ActualHours = (reader[3] is DBNull) ? 0 : (float)(reader[3] as decimal?) / 60
                        });
                    }
                    return projects;
                };
                projectsList = DBAccess.RunReader(query, funcProjects);


                //3 - get presences to worker
                Func<MySqlDataReader, List<_0_Presence>> funcPresences;

                foreach (_2_2_Project project in projectsList)
                {
                    query = $"SELECT pr.beginningTime, pr.endTime FROM projects_managment.presences pr WHERE pr.workerId = {worker.Id} AND pr.projectId = {project.Id} AND pr.endTime IS NOT NULL ";

                    funcPresences = (reader) =>
                    {
                        List<_0_Presence> presences = new List<_0_Presence>();
                        while (reader.Read())
                        {
                            presences.Add(new _0_Presence
                            {
                                BeginningTime = reader.GetDateTime(0),
                                EndTime = reader.GetDateTime(1)
                            });
                        }
                        return presences;
                    };

                    //add the presences
                    project.Presences = DBAccess.RunReader(query, funcPresences);

                    //add the projects
                    if (worker.Projects == null)
                        worker.Projects = new List<_2_2_Project>();
                    worker.Projects.Add(project);

                    worker.Projects.ForEach(p => worker.ActualHours += p.ActualHours);
                }
            }

            return workersList;
        }

        //GetTreeTeamLeaders
        public static List<_3_1_TeamLeader> GetTreeTeamLeaders()
        {
            string query;
            List<_3_1_TeamLeader> teamLeadersList;

            //1- get all team-leaders
            query = $"SELECT w.id, w.userName, w.firstName, w.lastName, w.eMail, w.phone FROM projects_managment.workers w WHERE statusId = 2 ";

            Func<MySqlDataReader, List<_3_1_TeamLeader>> funcTeamLeaders = (reader) =>
            {
                List<_3_1_TeamLeader> teamLeaders = new List<_3_1_TeamLeader>();

                while (reader.Read())
                {
                    teamLeaders.Add(new _3_1_TeamLeader
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5)
                    });
                }

                return teamLeaders;
            };

            teamLeadersList = DBAccess.RunReader(query, funcTeamLeaders);


            //2- get projects to team-leaders
            Func<MySqlDataReader, List<_3_2_Project>> funcProjects;
            List<_3_2_Project> projectsList;

            foreach (_3_1_TeamLeader teamLeader in teamLeadersList)
            {
                query = $"SELECT p.id, p.name, p.customerName, p.programhours, p.QAHours, p.UIUXHours, p.startDate, p.endDate FROM projects_managment.projects p WHERE p.projectManagerId = {teamLeader.Id} AND p.startDate <= DATE(NOW()) AND p.endDate >= DATE(NOW())";

                funcProjects = (reader) =>
                {
                    List<_3_2_Project> projects = new List<_3_2_Project>();
                    while (reader.Read())
                    {
                        projects.Add(new _3_2_Project
                        {
                            Id = reader.GetInt32(0),
                            Name = reader.GetString(1),
                            CustomerName = reader.GetString(2),
                            ProgramHours = (float?)reader.GetDecimal(3),
                            QAHours = (float?)reader.GetDecimal(4),
                            UIUXHours = (float?)reader.GetDecimal(5),
                            StartDate = reader.GetDateTime(6),
                            EndDate = reader.GetDateTime(7),
                            Departments = new List<_1_2_Department>()
                        });
                    }
                    return projects;
                };
                projectsList = DBAccess.RunReader(query, funcProjects);


                //3- get workers to projects
                Func<MySqlDataReader, List<_1_3_Worker>> funcWorkers;
                List<_1_3_Worker> workersList;

                foreach (_3_2_Project project in projectsList)
                {
                    query = $"SELECT w.id, w.userName, w.firstName, w.lastName, w.eMail, w.phone, w.statusId, pw.totalHours * 60 AS 'Required Minutes', " +
                        $"(SELECT SUM( TIMESTAMPDIFF(MINUTE, pr.beginningTime, pr.endTime)) FROM projects_managment.presences pr " +
                        $"WHERE pr.workerId=w.id AND pr.projectId = {project.Id}) AS 'Actual Minutes' " +
                        $"FROM projects_managment.workers w JOIN projects_managment.projectsworker pw ON w.id=pw.workerId " +
                        $"WHERE w.managerId = {teamLeader.Id} AND pw.projectId = {project.Id} ";

                    funcWorkers = (reader) =>
                    {
                        List<_1_3_Worker> workers = new List<_1_3_Worker>();
                        while (reader.Read())
                        {
                            workers.Add(new _1_3_Worker
                            {
                                Id = reader.GetInt32(0),
                                UserName = reader.GetString(1),
                                FirstName = reader.GetString(2),
                                LastName = reader.GetString(3),
                                EMail = reader.GetString(4),
                                Phone = reader.GetString(5),
                                StatusId = reader.GetInt32(6),
                                RequiredHours = (reader[7] is DBNull) ? 0 : (reader[7] as long?) / 60,
                                ActualHours = (reader[8] is DBNull) ? 0 : (float)(reader[8] as decimal?) / 60
                            });
                        }
                        return workers;
                    };
                    workersList = DBAccess.RunReader(query, funcWorkers);


                    //4- push the worker to the match department
                    foreach (_1_3_Worker worker in workersList)
                    {

                        _1_2_Department matchDepartment;
                        switch (worker.StatusId)
                        {
                            case 3:
                                {
                                    matchDepartment = project.Departments.Find(d => d.Name == "Program");
                                    if (matchDepartment == null)
                                    {
                                        //first time
                                        project.Departments.Add(new _1_2_Department
                                        {
                                            Name = "Program",
                                            Workers = new List<_1_3_Worker>(),
                                            RequiredHours = project.ProgramHours,
                                            ActualHours = 0
                                        });
                                        matchDepartment = project.Departments.Find(d => d.Name == "Program");
                                    }
                                    matchDepartment.Workers.Add(worker);
                                    matchDepartment.ActualHours += worker.ActualHours;
                                }
                                break;
                            case 4:
                                {
                                    matchDepartment = project.Departments.Find(d => d.Name == "QA");
                                    if (matchDepartment == null)
                                    {
                                        //first time
                                        project.Departments.Add(new _1_2_Department
                                        {
                                            Name = "QA",
                                            Workers = new List<_1_3_Worker>(),
                                            RequiredHours = project.QAHours,
                                            ActualHours = 0
                                        });
                                        matchDepartment = project.Departments.Find(d => d.Name == "QA");
                                    }
                                    matchDepartment.Workers.Add(worker);
                                    matchDepartment.ActualHours += worker.ActualHours;
                                }
                                break;
                            case 5:
                                {
                                    matchDepartment = project.Departments.Find(d => d.Name == "UI/UX");
                                    if (matchDepartment == null)
                                    {
                                        //first time
                                        project.Departments.Add(new _1_2_Department
                                        {
                                            Name = "UI/UX",
                                            Workers = new List<_1_3_Worker>(),
                                            RequiredHours = project.UIUXHours,
                                            ActualHours = 0
                                        });
                                        matchDepartment = project.Departments.Find(d => d.Name == "UI/UX");
                                    }
                                    matchDepartment.Workers.Add(worker);
                                    matchDepartment.ActualHours += worker.ActualHours;
                                }
                                break;
                            default:
                                break;
                        }
                    }
                }
                teamLeader.Projects = projectsList;
            }

            return teamLeadersList;
        }

    }
}

```
```csharp
using BOL;
using BOL.Help;
using DAL;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Mail;
using System.Timers;

namespace BLL
{
    public class LogicGlobal
    {

        //GetWorkerById
        public static Worker GetWorkerById(int workerId)
        {
            string query = $"SELECT id,userName,firstName,lastName,eMail,phone,statusId,managerId,totalHours " +
                $"FROM projects_managment.workers WHERE Id = { workerId} ";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    workers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        UserName = reader.GetString(1),
                        FirstName = reader.GetString(2),
                        LastName = reader.GetString(3),
                        EMail = reader.GetString(4),
                        Phone = reader.GetString(5),
                        StatusId = reader.GetInt32(6),
                        ManagerId = reader[7] as int?,
                        TotalHours = reader[8] as int?
                    });
                }
                return workers;
            };

            List<Worker> worker = DBAccess.RunReader(query, func);

            if (worker != null && worker.Count > 0)
                return worker[0];
            return null;
        }

        public static List<string> GetDirectorsMailAddress()
        {
            string query = $"SELECT eMail FROM projects_managment.workers WHERE statusId = " +
                $"(SELECT id FROM projects_managment.status WHERE statusName = 'director')";
            Func<MySqlDataReader, List<string>> func = (reader) =>
            {
                List<string> mailAddressList = new List<string>();
                while (reader.Read())
                {
                    mailAddressList.Add(reader.GetString(0));
                }
                return mailAddressList;
            };

            return DBAccess.RunReader(query, func);
        }

        public static Boolean SendMail(string getters, string subject, string message)
        {
            List<string> to = new List<string>();
            to.Add(getters);
            return SendMail(to, subject, message, null);
        }

        //SendMail - Get all required parametters (with list of all mail-addresses for sending), Try sending mail-message, Return if succeeded.
        public static Boolean SendMail(List<string> getters, string subject, string message, Worker sender)
        {
            MailMessage msg = new MailMessage();

            //msg.From = new MailAddress(sender.EMail);
            msg.From = new MailAddress("shtilimrishum2018@gmail.com");

            foreach (string item in getters)
            {
                msg.To.Add(item);
            }
            //mail subject
            if (sender != null)
                msg.Subject = subject + "from: " + sender.UserName + "    " + sender.EMail;
            msg.Subject = subject;

            //body
            msg.Body = message;
            SmtpClient client = new SmtpClient();
            client.UseDefaultCredentials = true;
            client.Host = "smtp.gmail.com";
            client.Port = 587;
            client.EnableSsl = true;
            client.DeliveryMethod = SmtpDeliveryMethod.Network;
            client.Credentials = new NetworkCredential("shtilimrishum2018@gmail.com", "0504190762");
            client.Timeout = 20000;
            try
            {
                client.Send(msg);
                return true;
            }
            catch (Exception)
            {
                return false;
            }
            finally
            {
                msg.Dispose();
            }
        }

        //AutoWarning - Send eMail to the workers (and their team-leaders) that  haven't finished their projects, and that projects are nearly \after the end-date
        public static void AutoWarning(object sender, ElapsedEventArgs args)
        {
            string query = $"SELECT w.eMail, p.name ,p.endDate FROM projects_managment.workers w" +
                $" JOIN projects_managment.projectsworker pw ON w.id = pw.workerId " +
                $"JOIN projects_managment.projects p ON p.id = pw.projectId " +
                $"WHERE p.endDate <= NOW() + INTERVAL 1 DAY" +
                $" UNION  " +
                $"SELECT w.eMail, p.name ,p.endDate FROM projects_managment.workers w " +
                $"JOIN projects_managment.projects p ON w.id = p.projectManagerId " +
                $"WHERE p.endDate <= NOW() + INTERVAL 1 DAY";

            Func<MySqlDataReader, List<AutoMail>> func = (reader) =>
            {
                List<AutoMail> autoMails = new List<AutoMail>();
                while (reader.Read())
                {
                    AutoMail autoMail = new AutoMail
                    {
                        EMail = reader.GetString(0),
                        Name = reader.GetString(1),
                        EndDate = Convert.ToDateTime(reader.GetString(2))
                    };

                    autoMails.Add(autoMail);
                }
                return autoMails;
            };

            List<AutoMail> autoMailsList = DBAccess.RunReader(query, func);

            if (autoMailsList != null)
                foreach (AutoMail autoMail in autoMailsList)
                {
                    SendMail(autoMail.EMail, $"the project {autoMail.Name} end in {autoMail.EndDate}!!", "    Please finished!!");
                }

        }

        //Notifications - Called from the WebApiConfig.Register
        public static void Notifications()
        {
            //call the AutoWarning when the server is on
            AutoWarning(null, null);

            //call the AutoWarning every day
            Timer timer = new Timer();
            timer.Interval = 60000 * 60 * 24; // once a day
            timer.Elapsed += new ElapsedEventHandler(AutoWarning);
            timer.Start();
        }

    }
}
```

### UIL
```csharp
using BLL;
using BOL;
using BOL.Help;
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;
using System.Net.Http.Formatting;
using System.Web.Http;

namespace Server_WebApi.Controllers
{
    public class UserController : ApiController
    {

        //curl -v -X POST -H "Content-type: application/json" -d "{\"Password\":\"12345\",\"EMail\":\"esty@gmail.com\"}"  http://localhost:60828/api/User/Login

        //curl -X GET -v http://localhost:60828/api/User/AutoLogin/10.10.10.10
        [HttpPost]
        [Route("api/User/AutoLogIn")]
        public HttpResponseMessage AutoLogIn([FromBody]dynamic details)
        {
            string ipa = details["ipAddress"];
            IPAddress ip;
            bool validateIp = IPAddress.TryParse(ipa, out ip);
            if (!validateIp)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, "The ip-address is not valid");
            }
            Worker worker;
            try
            {
                worker = LogicUser.AutoLogIn(ipa);
            }
            catch (Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e);
            }
            return Request.CreateResponse(HttpStatusCode.OK, worker);

        }

        [HttpPut]
        [Route("api/User/LogOut")]
        public HttpResponseMessage LogOut([FromBody]Worker worker)
        {
            //can not logout if worker has active task.
            if(!LogicUser.IsPassiveWorker(worker.Id))
                return new HttpResponseMessage(HttpStatusCode.BadRequest)
                {
                    Content = new ObjectContent<String>("You have an open task!", new JsonMediaTypeFormatter())
                };
            ModelState.Remove("worker.Password");
            if (ModelState.IsValid)
            {
                return (LogicUser.LogOut(worker)) ?
                    new HttpResponseMessage(HttpStatusCode.OK) :
                    new HttpResponseMessage(HttpStatusCode.BadRequest)
                    {
                        Content = new ObjectContent<String>("Can not update in DB", new JsonMediaTypeFormatter())
                    };
            }

            List<string> ErrorList = new List<string>();

            //if the code reached this part - the user is not valid
            foreach (var item in ModelState.Values)
                foreach (var err in item.Errors)
                    ErrorList.Add(err.ErrorMessage);

            return new HttpResponseMessage(HttpStatusCode.BadRequest)
            {
                Content = new ObjectContent<List<string>>(ErrorList, new JsonMediaTypeFormatter())
            };
        }

        // POST: api/Users
        [HttpPost]
        [Route("api/User/LogIn")]
        public HttpResponseMessage LogIn([FromBody]Login userLogin)
        {
            Worker worker;
            bool updatedIp;

            if (ModelState.IsValid)
            {
                try
                {
                    //get the match worker
                    worker = LogicUser.LogIn(userLogin.EMail, userLogin.Password);

                    if (worker == null)
                    {
                        return Request.CreateResponse(HttpStatusCode.OK, worker);

                    }
                    //if the worker has ipAddress and the worker send a request to update his ipAddress - return error response
                    if (worker.IpAddress != null)
                    {
                        return Request.CreateErrorResponse(HttpStatusCode.OK, "You are saved in another host, please first log-out");
                    }

                    if (userLogin.IpAddress != null)
                    {
                        //if there is another worker with that ipAddress - return error response
                        if ((LogicUser.AutoLogIn(userLogin.IpAddress)) != null)
                        {
                            return Request.CreateErrorResponse(HttpStatusCode.OK, "There is another saved-worker in that host, please first log-out");
                        }

                        //update the ipAddress
                        updatedIp = LogicUser.RememberMe(worker.Id, userLogin.IpAddress);
                        if (!updatedIp)
                        {
                            return Request.CreateErrorResponse(HttpStatusCode.OK, "Can't save the ip-address");
                        }

                        //get the correct worker
                        worker = LogicUser.LogIn(userLogin.EMail, userLogin.Password);
                    }
                }
                catch (Exception e)
                {
                    return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e);
                }
                return Request.CreateResponse(HttpStatusCode.OK, worker);
            }

            List<string> ErrorList = new List<string>();

            //if the code reached this part - the user is not valid
            foreach (var item in ModelState.Values)
                foreach (var err in item.Errors)
                    ErrorList.Add(err.ErrorMessage);

            return new HttpResponseMessage(HttpStatusCode.BadRequest)
            {
                Content = new ObjectContent<List<string>>(ErrorList, new JsonMediaTypeFormatter())
            };
        }

        // GetNotifications - Get workerId, Return his notifications
        [HttpGet]
        [Route("api/User/GetNotifications/{workerId}")]
        public HttpResponseMessage GetNotifications([FromUri]int workerId)
        {
            List<Notification> notifications;

            try
            {
                notifications = LogicUser.GetNotifications(workerId);
            }
            catch (Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e.Message);
            }

            return Request.CreateResponse(HttpStatusCode.OK, notifications);
        }

        //DeleteNotification - Delete notification by id, Return match message
        [HttpDelete]
        [Route("api/User/DeleteNotification/{notificationId}")]
        public HttpResponseMessage DeleteNotification([FromUri]int notificationId)
        {
            return (LogicUser.DeleteNotification(notificationId)) ?
                    new HttpResponseMessage(HttpStatusCode.OK) :
                    new HttpResponseMessage(HttpStatusCode.BadRequest)
                    {
                        Content = new ObjectContent<String>("Can not remove from DB", new JsonMediaTypeFormatter())
                    };
        }

        // GetProjectsToWorker - Get workerId, Return all the projects that the worker is working on them
        [HttpGet]
        [Route("api/User/GetProjectsToWorker/{workerId}")]
        public HttpResponseMessage GetProjectsToWorker([FromUri]int workerId)
        {
            List<Project> workerProjects;

            try
            {
                workerProjects = LogicUser.GetProjectsToWorker(workerId);
            }
            catch (Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e.Message);
            }

            return Request.CreateResponse(HttpStatusCode.OK, workerProjects);
        }

        //AddPresence - Get presence's details ,Add it to the D.B., Return the presenceId
        [HttpPost]
        [Route("api/User/AddPresence")]
        public HttpResponseMessage AddPresence([FromBody]Presence newPresence)
        {
            if (ModelState.IsValid)
            {
                int presenceId;
                try
                {
                    if (LogicUser.IsPassiveWorker(newPresence.WorkerId))
                    {
                        if (LogicUser.AddPresence(newPresence))
                        {

                            presenceId = LogicUser.GetCurrentPresenceId(newPresence);
                            return new HttpResponseMessage(HttpStatusCode.Created)
                            {
                                Content = new ObjectContent<Int32>(presenceId, new JsonMediaTypeFormatter())
                            };
                        }
                        else throw new Exception();
                    }
                    else return new HttpResponseMessage(HttpStatusCode.BadRequest)
                    {
                        Content = new ObjectContent<String>($"You are working now on another project.", new JsonMediaTypeFormatter())
                    };
                }
                catch (Exception e)
                {
                    return new HttpResponseMessage(HttpStatusCode.BadRequest)
                    {
                        Content = new ObjectContent<String>($"Can not add to DB - {e.Message}", new JsonMediaTypeFormatter())
                    };
                }
            };

            List<string> ErrorList = new List<string>();

            //if the code reached this part - the presence is not valid
            foreach (var item in ModelState.Values)
                foreach (var err in item.Errors)
                    ErrorList.Add(err.ErrorMessage);

            return new HttpResponseMessage(HttpStatusCode.BadRequest)
            {
                Content = new ObjectContent<List<string>>(ErrorList, new JsonMediaTypeFormatter())
            };

        }

        //UpdateEndTime - Get presenceId and endtime, Update in D.B. the endTime in the correct object (by the presenceId)
        [HttpPut]
        [Route("api/User/UpdateEndTime")]
        public HttpResponseMessage UpdateEndTime([FromBody]dynamic endTimeDetails)
        {
            int presenceId = endTimeDetails["presenceId"];
            DateTime endTime = endTimeDetails["endTime"];

            return (LogicUser.UpdateEndTime(presenceId, endTime)) ?
                new HttpResponseMessage(HttpStatusCode.OK) :
                new HttpResponseMessage(HttpStatusCode.BadRequest)
                {
                    Content = new ObjectContent<String>("Can not update in DB", new JsonMediaTypeFormatter())
                };
        }

        [HttpPost]
        [Route("api/User/SendEmailToManager")]
        public HttpResponseMessage SendEmailToManager([FromBody]dynamic data)
        {
            string subject;
            string message;
            int workerId;

            try
            {
                subject = data["subject"];
                message = data["message"];
                workerId = data["workerId"];
                Worker worker = LogicGlobal.GetWorkerById(workerId);
                List<string> managerMail = LogicGlobal.GetDirectorsMailAddress();
                bool isSended = LogicGlobal.SendMail(managerMail, subject, message, worker);
                if (isSended)
                    return Request.CreateResponse(HttpStatusCode.OK);
                else return Request.CreateErrorResponse(HttpStatusCode.BadRequest, "sending mail failed");
            }
            catch (Exception ex)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ex.Message);
            }

        }

        //curl -X GET -v http://localhost:60828/api/User/GetHoursStatusToWorker/4
        //GetHoursStatusToWorker - Get workerId, Return his hoursStatus. Each record contains the following columns:
        //workerId, projectId, RequiredHours, ActualHours.
        [HttpGet]
        [Route("api/User/GetHoursStatusToWorker/{workerId}")]
        public HttpResponseMessage GetHoursStatusToWorker([FromUri]int workerId)
        {
            List<HoursStatusW> workerProjects;

            try
            {
                workerProjects = LogicUser.GetHoursStatusToWorker(workerId);
            }
            catch (Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e.Message);
            }

            return Request.CreateResponse(HttpStatusCode.OK, workerProjects);
        }

    }
}
```
```csharp
using BLL;
using BOL;
using BOL.Help;
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;
using System.Net.Http.Formatting;
using System.Web.Http;

namespace Server_WebApi.Controllers
{
    public class TeamLeaderController : ApiController
    {
        //curl -X GET -v http://localhost:60828/api/TeamLeader/GetPresencesToTeamLeader/2
        //GetPresencesToTeamLeader - get projectManagerId, return all the presences to workers that are working in his staff
        //[HttpGet]
        //[Route("api/TeamLeader/GetPresencesToTeamLeader/{projectManagerId}")]
        //public HttpResponseMessage GetPresencesToTeamLeader([FromUri]int projectManagerId)
        //{
        //    return new HttpResponseMessage(HttpStatusCode.OK)
        //    {
        //        Content = new ObjectContent<List<Presence>>(LogicTeamLeader.GetPresencesToTeamLeader(projectManagerId), new JsonMediaTypeFormatter())
        //    };
        //}

        //curl -X GET -v http://localhost:60828/api/TeamLeader/GetProjectsToTeamLeader/2
        //GetProjectsToTeamLeader - Get team-leader-id, Return the projects he manages
        [HttpGet]
        [Route("api/TeamLeader/GetProjectsToTeamLeader/{teamLeaderId}")]
        public HttpResponseMessage GetProjectsToTeamLeader([FromUri]int teamLeaderId)
        {
            return new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new ObjectContent<List<Project>>(LogicTeamLeader.GetProjectsToTeamLeader(teamLeaderId), new JsonMediaTypeFormatter())
            };
        }

        //curl -X GET -v http://localhost:60828/api/TeamLeader/GetWorkersToTeamLeader/2
        //GetWorkersToTeamLeader - Get team-leader-id, Return the workers he manages
        [HttpGet]
        [Route("api/TeamLeader/GetWorkersToTeamLeader/{teamLeaderId}")]
        public HttpResponseMessage GetWorkersToTeamLeader([FromUri]int teamLeaderId)
        {
            return new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new ObjectContent<List<Worker>>(LogicTeamLeader.GetWorkersToTeamLeader(teamLeaderId), new JsonMediaTypeFormatter())
            };
        }

        //curl -X GET -v http://localhost:60828/api/TeamLeader/GetAllStatus
        //GetAllStatus - Return all statusses
        [HttpGet]
        [Route("api/TeamLeader/GetAllStatus")]
        public HttpResponseMessage GetAllStatus()
        {
            return new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new ObjectContent<List<Status>>(LogicTeamLeader.GetAllStatus(), new JsonMediaTypeFormatter())
            };
        }

        //לתקן הערה!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
        //curl -X GET -v http://localhost:60828/api/TeamLeader/GetHoursStatusToProject/13
        //GetHoursStatusToProject - Get projectId, Return its hoursStatus. Each record contains the following columns:
        //workerId, projectId, RequiredHours, ActualHours.
        [HttpGet]
        [Route("api/TeamLeader/GetHoursStatusToProject/{projectId}")]
        public HttpResponseMessage GetHoursStatusToProject([FromUri]int projectId)
        {
            List<HoursStatusTL> workerProjects;

            try
            {
                workerProjects = LogicTeamLeader.GetHoursStatusToProject(projectId);
            }
            catch (Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e.Message);
            }

            return Request.CreateResponse(HttpStatusCode.OK, workerProjects);
        }

        //UpdateTotalHoursToWorker - Update total-hours to worker, Return match message
        [HttpPut]
        [Route("api/TeamLeader/UpdateTotalHoursToWorker")]
        public HttpResponseMessage UpdateTotalHoursToWorker([FromBody]ProjectWorker projectWorker)
        {
            if (ModelState.IsValid)
            {
                return (LogicTeamLeader.UpdateTotalHoursToWorker(projectWorker)) ?
                    new HttpResponseMessage(HttpStatusCode.OK) :
                    new HttpResponseMessage(HttpStatusCode.BadRequest)
                    {
                        Content = new ObjectContent<String>("Can not update in DB", new JsonMediaTypeFormatter())
                    };
            };

            List<string> ErrorList = new List<string>();

            //if the code reached this part - the user is not valid
            foreach (var item in ModelState.Values)
                foreach (var err in item.Errors)
                    ErrorList.Add(err.ErrorMessage);

            return new HttpResponseMessage(HttpStatusCode.BadRequest)
            {
                Content = new ObjectContent<List<string>>(ErrorList, new JsonMediaTypeFormatter())
            };
        }

        //curl -X GET -v http://localhost:60828/api/TeamLeader/GetHoursStatusToWorker/4/2
        //GetHoursStatusToWorker - Get team-leader-id, Return the workers he manages
        [HttpGet]
        [Route("api/TeamLeader/GetHoursStatusToWorker/{workerId}/{teamLeaderId}")]
        public HttpResponseMessage GetHoursStatusToWorker([FromUri] int workerId, [FromUri]int teamLeaderId)
        {
            return new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new ObjectContent<List<HoursStatusW>>(LogicTeamLeader.GetHoursStatusToWorker(workerId, teamLeaderId), new JsonMediaTypeFormatter())
            };
        }

    }
}
```
```csharp
using BLL;
using BOL;
using BOL.Help;
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;
using System.Net.Http.Formatting;
using System.Web.Http;

namespace Server_WebApi.Controllers
{
    public class ManagerController : ApiController
    {
        //curl -X GET -v http://localhost:60828/api/Manager/GetAllStatus
        //GetAllStatus - Return all statusses
        [HttpGet]
        [Route("api/Manager/GetAllStatus")]
        public HttpResponseMessage GetAllStatus()
        {
            return new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new ObjectContent<List<Status>>(LogicManager.GetAllStatus(), new JsonMediaTypeFormatter())
            };
        }

        //curl -X GET -v http://localhost:60828/api/Manager/GetAllCompanyWorkers
        //GetAllCompanyWorkers - Return all company workers
        [HttpGet]
        [Route("api/Manager/GetAllCompanyWorkers")]
        public HttpResponseMessage GetAllCompanyWorkers()
        {
            return new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new ObjectContent<List<Worker>>(LogicManager.GetAllCompanyWorkers(), new JsonMediaTypeFormatter())
            };
        }

        //curl -X GET -v http://localhost:60828/api/Manager/GetAllWorkers
        //GetAllWorkers - Return all workers (program, UI/UX, QA)
        [HttpGet]
        [Route("api/Manager/GetAllWorkers")]
        public HttpResponseMessage GetAllWorkers()
        {
            return new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new ObjectContent<List<Worker>>(LogicManager.GetAllWorkers(), new JsonMediaTypeFormatter())
            };
        }

        //curl -X GET -v http://localhost:60828/api/Manager/GetAllTeamLeaders
        //GetAllManagers - Return all teamLeaders
        [HttpGet]
        [Route("api/Manager/GetAllTeamLeaders")]
        public HttpResponseMessage GetAllTeamLeaders()
        {
            return new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new ObjectContent<List<Worker>>(LogicManager.GetAllTeamLeaders(), new JsonMediaTypeFormatter())
            };
        }

        //curl -X GET -v http://localhost:60828/api/Manager/GetAllProjects
        //GetAllProjects - Return all proects
        [HttpGet]
        [Route("api/Manager/GetAllProjects")]
        public HttpResponseMessage GetAllProjects()
        {
            return new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new ObjectContent<List<Project>>(LogicManager.GetAllProjects(), new JsonMediaTypeFormatter())
            };
        }

        //curl -v -X POST -H "Content-type: application/json" -d "{\"name\":\"project-name\",\"customerName\":\"customer-name\",\"projectManagerId\":\"2\",\"programHours\":\"50\",\"QAHours\":\"30\",\"UIUXHours\":\"25\",\"startDate\":\"2018-09-09 03:44:00\",\"endDate\":\"2018-09-29 03:44:00\"}"  http://localhost:60828/api/Manager/AddProject
        //1
        //AddProject - with all parameter and managerId (without workers)
        //Return current projectId
        [HttpPost]
        [Route("api/Manager/AddProject")]
        public HttpResponseMessage AddProject([FromBody]Project newProject)
        {
            List<string> ErrorList = new List<string>();
            try
            {
                if (ModelState.IsValid)
                {
                    //2
                    bool created = LogicManager.AddProject(newProject);
                    if (created)
                    {
                        //3
                        //get current project id 
                        int currentProjectId = LogicManager.GetCurrentProjectId(newProject.Name);
                        //4
                        //return all the staff of the manager to add them to workerProject table.
                        List<Worker> workersToCurrentManager = LogicManager.WorkersToCurrentManager(newProject.ProjectManagerId);
                        //add staff to workerProject
                        foreach (Worker worker in workersToCurrentManager)
                        {
                            try
                            {
                                LogicManager.AddWorkerToProject(currentProjectId, worker.Id, 0);
                                //add to notification that they added to a new project
                                Notification notification = new Notification();
                                notification.Type = 1;
                                notification.WorkerId = worker.Id;
                                notification.Message = "You added to '" + newProject.Name + "' project ";
                                try
                                {
                                    bool succes = LogicManager.AddNotification(notification);
                                }
                                catch (Exception e)
                                {
                                    ErrorList.Add(e.Message);

                                }

                            }
                            catch (Exception e)
                            {
                                ErrorList.Add("Worker :" + worker.FirstName + " " + worker.LastName + "not added to workerProject" + e.Message);
                            }
                        }
                        //return the id of the new project
                        return Request.CreateResponse(HttpStatusCode.Created, currentProjectId);
                    }
                    else
                    {
                        return Request.CreateResponse(HttpStatusCode.BadRequest, "Can't add the project: " + newProject.Name);
                    }
                }
                foreach (var item in ModelState.Values)
                    foreach (var err in item.Errors)
                        ErrorList.Add(err.ErrorMessage);

                return Request.CreateResponse(HttpStatusCode.BadRequest, ErrorList);
            }
            catch (Exception ex)
            {
                return Request.CreateErrorResponse(HttpStatusCode.InternalServerError, ex.Message);
            }
        }

        //GetNotAllowedWorkersToProject - Get projectId, Return all workers are not allowed to work on that project
        [HttpGet]
        [Route("api/Manager/GetNotAllowedWorkersToProject/{projectId}")]
        public HttpResponseMessage GetNotAllowedWorkersToProject([FromUri]int projectId)
        {
            List<Worker> notAllowedWorkers;

            try
            {
                notAllowedWorkers = LogicManager.GetNotAllowedWorkersToProject(projectId);
            }
            catch (Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e.Message);
            }
            return Request.CreateResponse(HttpStatusCode.OK, notAllowedWorkers);
        }

        //AddOtherWorkersToProject - Get projectId and List<Worker>, Add the workers to yhat project, Return if succeedded
        [HttpPost]
        [Route("api/Manager/AddOtherWorkerstoProject/{projectId}")]
        public HttpResponseMessage AddOtherWorkersToProject([FromBody]List<Worker> otherWorkers, [FromUri]int projectId)
        {
            try
            {
                List<ObjectContent<string>> content = new List<ObjectContent<string>>();
                foreach (Worker worker in otherWorkers)
                {
                    if (!AddWorkerToProject(projectId, worker, 0))
                    {
                        content.Add(new ObjectContent<String>("Worker :" + worker.FirstName + " " + worker.LastName + "can not be added to workerProject", new JsonMediaTypeFormatter()));
                    }
                    else
                    {
                        string projectName = LogicManager.GetCurrentProjectName(projectId);
                        //add to notification that they added to a new project
                        Notification notification = new Notification();
                        notification.Type = 1;
                        notification.WorkerId = worker.Id;
                        notification.Message = "You added to " + projectName + " project ";
                        try
                        {
                            bool succes = LogicManager.AddNotification(notification);
                        }
                        catch (Exception e)
                        {
                            content.Add(new ObjectContent<String>(e.Message, new JsonMediaTypeFormatter()));
                        }
                    }
                }
                if (content.Count > 0)
                    return new HttpResponseMessage(HttpStatusCode.Created) { Content = new ObjectContent<String>(content.ToString(), new JsonMediaTypeFormatter()) };
                return new HttpResponseMessage(HttpStatusCode.Created);
            }
            catch (Exception)
            {
                return new HttpResponseMessage(HttpStatusCode.BadRequest)
                {
                    Content = new ObjectContent<String>("can't add workers to project", new JsonMediaTypeFormatter())
                };
            }
        }

        //AddWorker - Return match message
        [Route("api/Manager/AddWorker")]
        [HttpPost]
        public HttpResponseMessage AddWorker([FromBody]Worker newWorker)
        {
            if (ModelState.IsValid)
            {
                return (LogicManager.AddWorker(newWorker)) ?
                   new HttpResponseMessage(HttpStatusCode.Created) :
                   new HttpResponseMessage(HttpStatusCode.BadRequest)
                   {
                       Content = new ObjectContent<String>("Can not add to DB", new JsonMediaTypeFormatter())
                   };
            };

            List<string> ErrorList = new List<string>();

            //if the code reached this part - the user is not valid
            foreach (var item in ModelState.Values)
                foreach (var err in item.Errors)
                    ErrorList.Add(err.ErrorMessage);

            return new HttpResponseMessage(HttpStatusCode.BadRequest)
            {
                Content = new ObjectContent<List<string>>(ErrorList, new JsonMediaTypeFormatter())
            };
        }

        //GetWorkerById
        [HttpGet]
        [Route("api/Manager/GetWorkerById/{workerId}")]
        public HttpResponseMessage GetWorkerById([FromUri]int workerId)
        {
            try
            {
                Worker worker = LogicManager.GetWorkerById(workerId);
                return Request.CreateResponse(HttpStatusCode.OK, worker);
            }
            catch (Exception ex)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ex.Message);
            }
        }

        //UpdateWorkerDetails - Update worker-details, Return match message
        [HttpPut]
        [Route("api/Manager/UpdateWorkerDetails")]
        public HttpResponseMessage UpdateWorkerDetails([FromBody]Worker worker)
        {
            ModelState.Remove("worker.Password");
            if (ModelState.IsValid)
            {
                return (LogicManager.UpdateWorker(worker)) ?
                    new HttpResponseMessage(HttpStatusCode.OK) :
                    new HttpResponseMessage(HttpStatusCode.BadRequest)
                    {
                        Content = new ObjectContent<String>("Can not update in DB", new JsonMediaTypeFormatter())
                    };
            };

            List<string> ErrorList = new List<string>();

            //if the code reached this part - the user is not valid
            foreach (var item in ModelState.Values)
                foreach (var err in item.Errors)
                    ErrorList.Add(err.ErrorMessage);

            return new HttpResponseMessage(HttpStatusCode.BadRequest)
            {
                Content = new ObjectContent<List<string>>(ErrorList, new JsonMediaTypeFormatter())
            };
        }


        //DeleteWorker - Delete worker by id, Return match message
        [HttpDelete]
        [Route("api/Manager/DeleteWorker/{workerId}")]
        public HttpResponseMessage DeleteWorker([FromUri]int workerId)
        {
            return (LogicManager.RemoveWorker(workerId)) ?
                    new HttpResponseMessage(HttpStatusCode.OK) :
                    new HttpResponseMessage(HttpStatusCode.BadRequest)
                    {
                        Content = new ObjectContent<String>("Can not remove from DB", new JsonMediaTypeFormatter())
                    };
        }

        //AllowPremissionToWorker - Add worker to project, Return match message
        [HttpPost]
        [Route("api/Manager/AllowPremissionToWorker/{projectId}")]
        public HttpResponseMessage AllowPremissionToWorker([FromBody]Worker worker, [FromUri]int projectId)
        {
            if (AddWorkerToProject(projectId, worker, 0))
            {
                string projectName = LogicManager.GetCurrentProjectName(projectId);
                Notification notification = new Notification();
                notification.Type = 1;
                notification.WorkerId = worker.Id;
                notification.Message = "You added to " + projectName + " project ";
                try
                {
                    bool succes = LogicManager.AddNotification(notification);
                }
                catch (Exception e)
                {

                }

            }
            else
            {
                return new HttpResponseMessage(HttpStatusCode.BadRequest)
                {
                    Content = new ObjectContent<String>("Worker :" + worker.FirstName + " " + worker.LastName + "can not be added to project: " + projectId, new JsonMediaTypeFormatter())
                };
            }
            return new HttpResponseMessage(HttpStatusCode.Created);
        }

        //AddWorkerToProject - Get projectId and worker, Return if succeeded adding that worker to the correct project
        private bool AddWorkerToProject(int projectId, Worker worker, int totalHours)
        {
            return (LogicManager.AddWorkerToProject(projectId, worker.Id, totalHours));
        }

        //UpdateTeamLeaderToWorker - Update teamLeaderId to the correct worker
        [HttpPut]
        [Route("api/Manager/UpdateTeamLeaderToWorker/{workerId}")]
        public HttpResponseMessage UpdateTeamLeaderToWorker([FromUri]int workerId, [FromBody]int teamLeaderId)
        {
            if (LogicManager.UpdateTeamLeaderToWorker(workerId, teamLeaderId))
            {
                //Notification
                string teamLeaderName = LogicManager.GetTeamLeaderName(teamLeaderId);
                Notification notification = new Notification();
                notification.Type = 2;
                notification.WorkerId = workerId;
                notification.Message = "Your team leader has been replaced. Your new team leader is :" + teamLeaderName;
                try
                {
                    bool succes = LogicManager.AddNotification(notification);
                }
                catch (Exception e)
                {

                }
            }
            else
            {
                return new HttpResponseMessage(HttpStatusCode.BadRequest)
                {
                    Content = new ObjectContent<String>("Can not update in DB", new JsonMediaTypeFormatter())
                };
            }
            return new HttpResponseMessage(HttpStatusCode.OK);
        }


        //----------Trees-----------------

        //GetTreeProjects
        [HttpGet]
        [Route("api/Manager/GetTreeProjects")]
        public HttpResponseMessage GetTreeProjects()
        {
            List<_1_1_Project> TreeProjects;
            try
            {
                TreeProjects = LogicManager.GetTreeProjects();
            }
            catch (Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e.Message);
            }
            return Request.CreateResponse(HttpStatusCode.OK, TreeProjects);
        }

        //GetTreeWorkers
        [HttpGet]
        [Route("api/Manager/GetTreeWorkers")]
        public HttpResponseMessage GetTreeWorkers()
        {
            List<_2_1_Worker> TreeWorkers;
            try
            {
                TreeWorkers = LogicManager.GetTreeWorkers();
            }
            catch (Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e.Message);
            }
            return Request.CreateResponse(HttpStatusCode.OK, TreeWorkers);
        }

        //GetTreeTeamLeaders
        [HttpGet]
        [Route("api/Manager/GetTreeTeamLeaders")]
        public HttpResponseMessage GetTreeTeamLeaders()
        {
            List<_3_1_TeamLeader> TreeTeamLeaders;
            try
            {
                TreeTeamLeaders = LogicManager.GetTreeTeamLeaders();
            }
            catch (Exception e)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, e.Message);
            }
            return Request.CreateResponse(HttpStatusCode.OK, TreeTeamLeaders);
        }
    }
}
```

# Test api with `curl`

### Get Request
```
curl -X GET -v http://localhost:59628/api/getAllWorkers
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getAllWorkers HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldEFsbFdvcmtlcnM=?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:07:20 GMT
< Content-Length: 769
<
[{"Id":1,"Name":"manager","UserName":"mm","Password":"","JobId":1,"EMail":"manag
@gmail.com","ManagerId":null},{"Id":21,"Name":"TeamLeader","UserName":"tt","Pass
word":"","JobId":2,"EMail":"team@gmail.com","ManagerId":1},{"Id":22,"Name":"Work
er1","UserName":"ww1","Password":"","JobId":3,"EMail":"worker1@gmail.com","Manag
erId":21},{"Id":23,"Name":"Worker2","UserName":"ww2","Password":"","JobId":4,"EM
ail":"worker2@gmail.com","ManagerId":21},{"Id":24,"Name":"Worker3","UserName":"w
w3","Password":"","JobId":5,"EMail":"worker3@gmail.com","ManagerId":21},{"Id":25
,"Name":"Worker4","UserName":"ww4","Password":"","JobId":3,"EMail":"worker4@gmai
l.com","ManagerId":21},{"Id":27,"Name":"Gila","UserName":"gggg","Password":"","J
obId":4,"EMail":"safdsa@fdsa.fc","ManagerId":21}]* Connection #0 to host localho
st left intact
```
```
curl -X GET -v http://localhost:59628/api/GetPresence
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/GetPresence HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXEdldFByZXNlbmNl?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:08:57 GMT
< Content-Length: 5501
<
[{"WorkerName":"Worker1","ProjectName":"pr1","Date":"2018-11-27T00:00:00","Start
":"08:30:48","End":"15:28:56"},{"WorkerName":"Worker1","ProjectName":"pr1","Date
":"2018-11-28T00:00:00","Start":"11:23:50","End":"11:24:01"},{"WorkerName":"Work
er1","ProjectName":"pr2","Date":"2018-11-28T00:00:00","Start":"08:02:57","End":"
14:15:59"},{"WorkerName":"Worker1","ProjectName":"pr3","Date":"2018-11-29T00:00:
00","Start":"09:33:00","End":"17:45:01"},{"WorkerName":"Worker2","ProjectName":"
pr1","Date":"2018-11-28T00:00:00","Start":"10:29:11","End":"16:28:11"},{"WorkerN
ame":"Worker2","ProjectName":"pr2","Date":"2018-11-27T00:00:00","Start":"07:29:1
3","End":"16:46:15"},{"WorkerName":"Worker2","ProjectName":"pr3","Date":"2018-11
-29T00:00:00","Start":"08:09:16","End":"15:27:17"},{"WorkerName":"Worker3","Proj
ectName":"pr1","Date":"2018-11-29T00:00:00","Start":"08:15:29","End":"17:45:29"}
,{"WorkerName":"Worker3","ProjectName":"pr2","Date":"2018-11-28T00:00:00","Start
":"09:48:30","End":"13:06:31"},{"WorkerName":"Worker3","ProjectName":"pr3","Date
":"2018-11-27T00:00:00","Start":"10:29:32","End":"17:39:32"},{"WorkerName":"Work
er4","ProjectName":"pr1","Date":"2018-11-27T00:00:00","Start":"08:50:44","End":"
15:29:44"},{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11-27T00:00:
00","Start":"12:18:12","End":"12:18:19"},{"WorkerName":"Worker4","ProjectName":"
pr1","Date":"2018-11-27T00:00:00","Start":"14:23:45","End":"14:26:33"},{"WorkerN
ame":"Worker4","ProjectName":"pr1","Date":"2018-11-27T00:00:00","Start":"14:26:2
8","End":"14:26:33"},{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11
-27T00:00:00","Start":"14:26:43","End":"14:28:01"},{"WorkerName":"Worker4","Proj
ectName":"pr1","Date":"2018-11-27T00:00:00","Start":"14:27:36","End":"14:28:01"}
,{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11-27T00:00:00","Start
":"14:33:38","End":"14:33:50"},{"WorkerName":"Worker4","ProjectName":"pr1","Date
":"2018-11-27T00:00:00","Start":"14:34:21","End":"14:34:35"},{"WorkerName":"Work
er4","ProjectName":"pr1","Date":"2018-11-27T00:00:00","Start":"14:35:26","End":"
14:35:46"},{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11-27T00:00:
00","Start":"14:37:23","End":"14:37:27"},{"WorkerName":"Worker4","ProjectName":"
pr1","Date":"2018-11-27T00:00:00","Start":"14:37:42","End":"14:37:48"},{"WorkerN
ame":"Worker4","ProjectName":"pr1","Date":"2018-11-27T00:00:00","Start":"14:37:5
2","End":"14:38:01"},{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11
-28T00:00:00","Start":"08:37:41","End":"08:38:46"},{"WorkerName":"Worker4","Proj
ectName":"pr1","Date":"2018-11-28T00:00:00","Start":"08:39:03","End":"08:57:38"}
,{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11-28T00:00:00","Start
":"08:39:56","End":"08:57:38"},{"WorkerName":"Worker4","ProjectName":"pr1","Date
":"2018-11-28T00:00:00","Start":"08:41:00","End":"08:57:38"},{"WorkerName":"Work
er4","ProjectName":"pr1","Date":"2018-11-28T00:00:00","Start":"08:43:27","End":"
08:57:38"},{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11-28T00:00:
00","Start":"08:44:23","End":"08:57:38"},{"WorkerName":"Worker4","ProjectName":"
pr1","Date":"2018-11-28T00:00:00","Start":"08:57:32","End":"08:57:38"},{"WorkerN
ame":"Worker4","ProjectName":"pr1","Date":"2018-11-28T00:00:00","Start":"08:57:4
2","End":"08:57:48"},{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11
-28T00:00:00","Start":"08:57:49","End":"09:05:02"},{"WorkerName":"Worker4","Proj
ectName":"pr1","Date":"2018-11-28T00:00:00","Start":"08:58:14","End":"09:05:02"}
,{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11-28T00:00:00","Start
":"09:04:57","End":"09:05:02"},{"WorkerName":"Worker4","ProjectName":"pr1","Date
":"2018-11-28T00:00:00","Start":"09:05:14","End":"09:16:08"},{"WorkerName":"Work
er4","ProjectName":"pr1","Date":"2018-11-28T00:00:00","Start":"09:05:36","End":"
09:16:08"},{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11-28T00:00:
00","Start":"09:15:47","End":"09:16:08"},{"WorkerName":"Worker4","ProjectName":"
pr1","Date":"2018-11-28T00:00:00","Start":"09:16:53","End":"09:17:05"},{"WorkerN
ame":"Worker4","ProjectName":"pr1","Date":"2018-11-28T00:00:00","Start":"09:19:2
0","End":"09:19:26"},{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11
-28T00:00:00","Start":"09:19:41","End":"09:19:52"},{"WorkerName":"Worker4","Proj
ectName":"pr1","Date":"2018-11-28T00:00:00","Start":"09:20:04","End":"09:20:09"}
,{"WorkerName":"Worker4","ProjectName":"pr1","Date":"2018-11-28T00:00:00","Start
":"09:25:11","End":"09:25:20"},{"WorkerName":"Worker4","ProjectName":"pr1","Date
":"2018-11-28T00:00:00","Start":"09:25:25","End":"09:25:26"},{"WorkerName":"Work
er4","ProjectName":"pr2","Date":"2018-11-28T00:00:00","Start":"09:12:45","End":"
14:59:48"},{"WorkerName":"Worker4","ProjectName":"pr2","Date":"2018-11-28T00:00:
00","Start":"09:20:37","End":"09:20:47"},{"WorkerName":"Worker4","ProjectName":"
pr2","Date":"2018-11-28T00:00:00","Start":"09:22:03","End":"09:22:12"},{"WorkerN
ame":"Worker4","ProjectName":"pr2","Date":"2018-11-28T00:00:00","Start":"09:22:2
3","End":"09:22:29"},{"WorkerName":"Worker4","ProjectName":"pr2","Date":"2018-11
-28T00:00:00","Start":"09:23:48","End":"09:23:53"},{"WorkerName":"Worker4","Proj
ectName":"pr2","Date":"2018-11-28T00:00:00","Start":"09:24:00","End":"09:24:03"}
,{"WorkerName":"Worker4","ProjectName":"pr3","Date":"2018-11-29T00:00:00","Start
":"10:03:50","End":"15:49:53"},{"WorkerName":"Worker4","ProjectName":"pr3","Date
":"2018-11-30T00:00:00","Start":"07:29:54","End":"15:29:57"}]* Connection #0 to
host localhost left intact
```
```
curl -X GET -v http://localhost:59628/api/getAllManagers
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getAllManagers HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldEFsbE1hbmFnZXJz?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:11:39 GMT
< Content-Length: 219
<
[{"Id":1,"Name":"manager","UserName":"mm","Password":"","JobId":1,"EMail":"manag
@gmail.com","ManagerId":null},{"Id":21,"Name":"TeamLeader","UserName":"tt","Pass
word":"","JobId":2,"EMail":"team@gmail.com","ManagerId":1}]* Connection #0 to ho
st localhost left intact
```
```
curl -X GET -v http://localhost:59628/api/getProjectDeatails/21
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getProjectDeatails/21 HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldFByb2plY3REZWF0YWlsc1wyMQ==?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:13:21 GMT
< Content-Length: 686
<
[{"Id":7,"Name":"pr1","TeamLeaderId":21,"Customer":"aaa","DevelopHours":280,"QAH
ours":130,"UiUxHours":45,"StartDate":"2018-11-27T00:00:00","EndDate":"2019-01-26
T00:00:00"},{"Id":8,"Name":"pr2","TeamLeaderId":21,"Customer":"aaa","DevelopHour
s":480,"QAHours":56,"UiUxHours":210,"StartDate":"2018-12-12T00:00:00","EndDate":
"2019-03-30T00:00:00"},{"Id":9,"Name":"pr3","TeamLeaderId":21,"Customer":"ddd","
DevelopHours":250,"QAHours":45,"UiUxHours":67,"StartDate":"2018-11-30T00:00:00",
"EndDate":"2019-10-26T00:00:00"},{"Id":10,"Name":"tr1","TeamLeaderId":21,"Custom
er":"nnn","DevelopHours":300,"QAHours":250,"UiUxHours":100,"StartDate":"2018-02-
02T00:00:00","EndDate":"2018-07-07T00:00:00"}]* Connection #0 to host localhost
left intact
```
```
curl -X GET -v http://localhost:59628/api/getWorkersDeatails/21
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getWorkersDeatails/21 HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldFdvcmtlcnNEZWF0YWlsc1wyMQ==?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:14:36 GMT
< Content-Length: 551
<
[{"Id":22,"Name":"Worker1","UserName":"ww1","Password":"","JobId":3,"EMail":"wor
ker1@gmail.com","ManagerId":21},{"Id":23,"Name":"Worker2","UserName":"ww2","Pass
word":"","JobId":4,"EMail":"worker2@gmail.com","ManagerId":21},{"Id":24,"Name":"
Worker3","UserName":"ww3","Password":"","JobId":5,"EMail":"worker3@gmail.com","M
anagerId":21},{"Id":25,"Name":"Worker4","UserName":"ww4","Password":"","JobId":3
,"EMail":"worker4@gmail.com","ManagerId":21},{"Id":27,"Name":"Gila","UserName":"
gggg","Password":"","JobId":4,"EMail":"safdsa@fdsa.fc","ManagerId":21}]* Connect
ion #0 to host localhost left intact
```
```
curl -X GET -v http://localhost:59628/api/getWorkersHours/7
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getWorkersHours/7 HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldFdvcmtlcnNIb3Vyc1w3?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:16:51 GMT
< Content-Length: 275
<
[{"Name":"Gila","Hours":"0:0","AllocatedHours":0.0},{"Name":"Worker1","Hours":"6
:58","AllocatedHours":25.0},{"Name":"Worker2","Hours":"5:59","AllocatedHours":46
.0},{"Name":"Worker3","Hours":"9:30","AllocatedHours":70.0},{"Name":"Worker4","H
ours":"8:43","AllocatedHours":6.4}]* Connection #0 to host localhost left intact
```
```
curl -X GET -v http://localhost:59628/api/getWorkerHours/21/22
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getWorkerHours/21/22 HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldFdvcmtlckhvdXJzXDIxXDIy?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:18:19 GMT
< Content-Length: 248
<
[{"Id":48,"Name":"pr1","AllocatedHours":25.0,"Hours":"06:58:19"},{"Id":52,"Name"
:"pr2","AllocatedHours":30.0,"Hours":"06:13:02"},{"Id":56,"Name":"pr3","Allocate
dHours":18.0,"Hours":"08:12:01"},{"Id":60,"Name":"tr1","AllocatedHours":0.0,"Hou
rs":""}]* Connection #0 to host localhost left intact
```
```
curl -X GET -v http://localhost:59628/api/getWorkerDetails/22
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getWorkerDetails/22 HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldFdvcmtlckRldGFpbHNcMjI=?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:20:40 GMT
< Content-Length: 110
<
{"Id":22,"Name":"Worker1","UserName":"ww1","Password":"","JobId":3,"EMail":"work
er1@gmail.com","ManagerId":21}* Connection #0 to host localhost left intact
```
```
curl -X GET -v http://localhost:59628/api/getProject/7
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getProject/7 HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldFByb2plY3RcNw==?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:21:56 GMT
< Content-Length: 2
<
[]* Connection #0 to host localhost left intact
```
### Put Request (not valid data)
```
curl -v -X PUT -H "Content-type: application/json" -d "{\"Id\":\"7\",\"Name\":\"Malki\", \"UserName\":\"ggggg\",\"Password\":\"mmmggg\" , \"JobId\":\"3\",\"EMail\":\"sjafjkl@df.vaf\", \"ManagerId\":\"21\"}"  http://localhost:59628/api/UpdateWorker
```
```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> PUT /api/UpdateWorker HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 122
>
* upload completely sent off: 122 out of 122 bytes
< HTTP/1.1 400 Bad Request
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXFVwZGF0ZVdvcmtlcg==?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:31:54 GMT
< Content-Length: 22
<
"Can not update in DB"* Connection #0 to host localhost left intact
```
```
curl -v -X PUT -H "Content-type: application/json" -d "{\"projectWorkerId\":\"1\",\"numHours\":\"30\"}"  http://localhost:59628/api/updateWorkerHours
```
```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> PUT /api/updateWorkerHours HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 39
>
* upload completely sent off: 39 out of 39 bytes
< HTTP/1.1 400 Bad Request
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXHVwZGF0ZVdvcmtlckhvdXJz?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:38:47 GMT
< Content-Length: 29
<
"Can not update in Data Base"* Connection #0 to host localhost left intact
```
### Put Request (valid data)
```
curl -v -X PUT -H "Content-type: application/json" -d "{\"Id\":\"27\",\"Name\":\"Malki\", \"UserName\":\"1ggg\",\"Password\":\"mmmggg\" , \"JobId\":\"3\",\"EMail\":\"sjafjkl@df.vaf\", \"ManagerId\":\"21\"}"  http://localhost:59628/api/UpdateWorker
```
```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> PUT /api/UpdateWorker HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 122
>
* upload completely sent off: 122 out of 122 bytes
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXFVwZGF0ZVdvcmtlcg==?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:36:40 GMT
< Content-Length: 0
<
* Connection #0 to host localhost left intact
```
```
curl -v -X PUT -H "Content-type: application/json" -d "{\"projectWorkerId\":\"48\",\"numHours\":\"30\"}"  http://localhost:59628/api/updateWorkerHours
```
```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> PUT /api/updateWorkerHours HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 40
>
* upload completely sent off: 40 out of 40 bytes
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXHVwZGF0ZVdvcmtlckhvdXJz?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:40:00 GMT
< Content-Length: 0
<
* Connection #0 to host localhost left intact
```

### Post Request (not valid data)
```
curl -v -X POST -H "Content-type: application/json" -d "{\"UserName\":\"DDD\", \"Password\":\"444444\"}"  http://localhost:59628/api/login
```
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> POST /api/login HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 39
>
* upload completely sent off: 39 out of 39 bytes
< HTTP/1.1 400 Bad Request
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGxvZ2lu?=
< X-Powered-By: ASP.NET
< Date: Wed, 28 Nov 2018 09:50:14 GMT
< Content-Length: 16
<
"Can not log in"* Connection #0 to host localhost left intact
```
```
curl -v -X POST -H "Content-type: application/json" -d "{\"Name\":\"tryProject\", \"Customer\":\"nnn\",\"TeamLeaderId\":\"11\" , \"DevelopHours\":\"300\",\"QAHours\":\"250\", \"UiUxHours\":\"100\",\"StartDate\":\"2018-02-02\",\"EndDate\":\"2018-07-07\"}"  http://localhost:59628/api/addProject
```
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
*   Trying 127.0.0.1...
* TCP_NODELAY set
* connect to ::1 port 59628 failed: Connection refused
* connect to 127.0.0.1 port 59628 failed: Connection refused
* Failed to connect to localhost port 59628: Connection refused
* Closing connection 0
curl: (7) Failed to connect to localhost port 59628: Connection refused
```
```
curl -v -X POST -H "Content-type: application/json" -d "{\"Name\":\"Gila\",\"UserName\":\"gggg\",\
"Password\":\"gggggg\",\"JobId\":\"4\",\"EMail\":\"safdsa@fdsaf\",\"ManagerId\":\"11\"}"  http://localhost:59628/api/addWorker
```
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> POST /api/addWorker HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 105
>
* upload completely sent off: 105 out of 105 bytes
< HTTP/1.1 400 Bad Request
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGFkZFdvcmtlcg==?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 06:42:53 GMT
< Content-Length: 19
<
"Can not add to DB"* Connection #0 to host localhost left intact
```
```
curl -v -X POST -H "Content-type: application/json" -d "{\"idProjectWorker\":\"7\",\"hour\":\"8\","isFirst\":\"true\"}"  http://localhost:59628/api/updateStartHour
```
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> POST /api/updateHours HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 21
>
* upload completely sent off: 21 out of 21 bytes
< HTTP/1.1 404 Not Found
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXHVwZGF0ZUhvdXJz?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 06:48:36 GMT
< Content-Length: 196
<
{"Message":"No HTTP resource was found that matches the request URI 'http://loca
lhost:59628/api/updateHours'.","MessageDetail":"No type was found that matches t
he controller named 'updateHours'."}* Connection #0 to host localhost left intact
```
```
curl -v -X POST -H "Content-type: application/json" -d "{\"sub\":\"malky8895\",\"body\":\"ddd\",\"id\":\"9\"}"  http://localhost:59628/api/SendMsg
```
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> POST /api/SendMsg HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 41
>
* upload completely sent off: 41 out of 41 bytes
< HTTP/1.1 500 Internal Server Error
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXFNlbmRNc2c=?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:04:54 GMT
< Content-Length: 2634
```

### Post Request (valid data)
```
curl-7.61.0-win64-mingw\bin>curl -v -X POST -H "Content-type: application/json" -d "{\"UserName\":\"mm\", \"Password\":\"6773ea887f0e3f1c34b01936aaf9687b16a04c6f9e65e4afbfce7bb7f76b0857\"}"  http://localhost:59628/api/login
```
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> POST /api/login HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 96
>
* upload completely sent off: 96 out of 96 bytes
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGxvZ2lu?=
< X-Powered-By: ASP.NET
< Date: Wed, 28 Nov 2018 09:55:38 GMT
< Content-Length: 108
<
{"Id":1,"Name":"manager","UserName":"mm","Password":"","JobId":1,"EMail":"manag@
gmail.com","ManagerId":null}* Connection #0 to host localhost left intact
```
```
C:\Users\administrator.NB\Desktop\curl-7.61.0-win64-mingw\bin>curl -v -X POST -H
 "Content-type: application/json" -d "{\"Name\":\"tr1\", \"Customer\":\"nnn\",\"TeamLeaderId\":\"21\" 
 , \"DevelopHours\":\"300\",\"QAHours\":\"250\", \"UiUxHours\":\"100\",\"StartDate\":\"2018-02-02\",\"EndDate\":\"2018-07-07\"}"  http://localhost:59628/api/addProject
```
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> POST /api/addProject HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 158
>
* upload completely sent off: 158 out of 158 bytes
< HTTP/1.1 201 Created
< Cache-Control: no-cache
< Pragma: no-cache
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGFkZFByb2plY3Q=?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 06:40:12 GMT
< Content-Length: 0
<
* Connection #0 to host localhost left intact
```
```
C:\Users\administrator.NB\Desktop\curl-7.61.0-win64-mingw\bin>curl -v -X POST -H "Content-type: application/json" -d "{\"Name\":\"Gila\",\"UserName\":\"gggg\",\
"Password\":\"gggggg\",\"JobId\":\"4\",\"EMail\":\"safdsa@fdsa.fc\",\"ManagerId\":\"21\"}"  http://localhost:59628/api/addWorker
```
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> POST /api/addWorker HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 107
>
* upload completely sent off: 107 out of 107 bytes
< HTTP/1.1 201 Created
< Cache-Control: no-cache
< Pragma: no-cache
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGFkZFdvcmtlcg==?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 06:46:11 GMT
< Content-Length: 0
<
* Connection #0 to host localhost left intact
```
```
curl -v -X POST -H "Content-type: application/json" -d "{\"idProjectWorker\":\"22\",\"hour\":\"8\","isFirst\":\"true\"}"  http://localhost:59628/api/updateStartHour
```
```
```
```
curl -v -X POST -H "Content-type: application/json" -d "{\"sub\":\"malky8895\",\"body\":\"ddd\",\"id\":\"22\"}"  http://localhost:59628/api/SendMsg
```
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> POST /api/SendMsg HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
> Content-type: application/json
> Content-Length: 42
>
* upload completely sent off: 42 out of 42 bytes
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXFNlbmRNc2c=?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:03:36 GMT
< Content-Length: 0
<
* Connection #0 to host localhost left intact
```
### Delete Request (not valid data)
```
curl -X DELETE -v http://localhost:59628/api/deleteWorker/8
```
```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> DELETE /api/deleteWorker/8 HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 400 Bad Request
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGRlbGV0ZVdvcmtlclw4?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:43:18 GMT
< Content-Length: 24
<
"Can not remove from DB"* Connection #0 to host localhost left intact
```
### Delete Request (valid data)
```
curl -X GET -v http://localhost:59628/api/getAllWorkers
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getAllWorkers HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldEFsbFdvcmtlcnM=?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:44:47 GMT
< Content-Length: 770
<
[{"Id":1,"Name":"manager","UserName":"mm","Password":"","JobId":1,"EMail":"manag
@gmail.com","ManagerId":null},{"Id":21,"Name":"TeamLeader","UserName":"tt","Pass
word":"","JobId":2,"EMail":"team@gmail.com","ManagerId":1},{"Id":22,"Name":"Work
er1","UserName":"ww1","Password":"","JobId":3,"EMail":"worker1@gmail.com","Manag
erId":21},{"Id":23,"Name":"Worker2","UserName":"ww2","Password":"","JobId":4,"EM
ail":"worker2@gmail.com","ManagerId":21},{"Id":24,"Name":"Worker3","UserName":"w
w3","Password":"","JobId":5,"EMail":"worker3@gmail.com","ManagerId":21},{"Id":25
,"Name":"Worker4","UserName":"ww4","Password":"","JobId":3,"EMail":"worker4@gmai
l.com","ManagerId":21},{"Id":27,"Name":"Malki","UserName":"1ggg","Password":"","
JobId":3,"EMail":"sjafjkl@df.vaf","ManagerId":21}]* Connection #0 to host localh
ost left intact
```
```
curl -X DELETE -v http://localhost:59628/api/deleteWorker/27
```
```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> DELETE /api/deleteWorker/27 HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGRlbGV0ZVdvcmtlclwyNw==?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:45:46 GMT
< Content-Length: 0
<
* Connection #0 to host localhost left intact
```
```
curl -X GET -v http://localhost:59628/api/getAllWorkers
```
```
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 59628 (#0)
> GET /api/getAllWorkers HTTP/1.1
> Host: localhost:59628
> User-Agent: curl/7.61.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Pragma: no-cache
< Content-Type: application/json; charset=utf-8
< Expires: -1
< Server: Microsoft-IIS/10.0
< X-AspNet-Version: 4.0.30319
< X-SourceFiles: =?UTF-8?B?QzpcVXNlcnNcc2VsZGF0XERvY3VtZW50c1xHaXRIdWJcVGFza01hb
mFnbWVudDJcVGFza01hbmFnbWVudFxVSUxcYXBpXGdldEFsbFdvcmtlcnM=?=
< X-Powered-By: ASP.NET
< Date: Thu, 29 Nov 2018 07:46:33 GMT
< Content-Length: 663
<
[{"Id":1,"Name":"manager","UserName":"mm","Password":"","JobId":1,"EMail":"manag
@gmail.com","ManagerId":null},{"Id":21,"Name":"TeamLeader","UserName":"tt","Pass
word":"","JobId":2,"EMail":"team@gmail.com","ManagerId":1},{"Id":22,"Name":"Work
er1","UserName":"ww1","Password":"","JobId":3,"EMail":"worker1@gmail.com","Manag
erId":21},{"Id":23,"Name":"Worker2","UserName":"ww2","Password":"","JobId":4,"EM
ail":"worker2@gmail.com","ManagerId":21},{"Id":24,"Name":"Worker3","UserName":"w
w3","Password":"","JobId":5,"EMail":"worker3@gmail.com","ManagerId":21},{"Id":25
,"Name":"Worker4","UserName":"ww4","Password":"","JobId":3,"EMail":"worker4@gmai
l.com","ManagerId":21}]* Connection #0 to host localhost left intact
```


## WinForms + Angular
### Home page
![picture](step2.png)
![picture](step2.1.png)   
### Manager page
![picture](step3.png)   
![picture](step4.png) 
![picture](step4.1.png)  
![picture](step4.2.png)  
![picture](step4.3.png)  
![picture](step4.4.png)  
![picture](step4.5.png)  
### Team-leader page
![picture](step5.png)   
![picture](step6.png)  
![picture](step7.png) 
### Worker page
![picture](step12.png) 
![picture](step10.png) 
![picture](step13.png)



