### TUTORIAL PRODUCE AN API WITH .NET CORE V. 3.11

1. #### Create Project with tempalate "API Web And Console ASP.Net Core"
    ![alt text](./img/Screen%20Shot%202022-07-04%20at%2016.01.24.png)
2. ####  Select Framework Target 3.1
    ![alt text](./img/Screen%20Shot%202022-07-04%20at%2018.48.55.png)
3. #### Typing your Project names
    ![alt text](./img/Screen%20Shot%202022-07-04%20at%2018.49.26.png)
4. #### Create Model Folder and Model Files as Entity Schema for automatically migrations to database sql server.

    * View of Entity Relations Diagram Structure.
    
       ![alt text](./img/Screen%20Shot%202022-07-04%20at%2018.56.54.png)

    From this diagram we can assumptions the relations type entyty is "one two many" , one branch can using by many employees. if we're look from json side, we can conclusing the branch table object can be have much emplooyes and the employees table can have one by one branch data. 
    * And then, if we want implment this schema on the models we can try write anotaions on models like that picture:

        ![alt text](./img/Screen%20Shot%202022-07-04%20at%2019.40.12.png)

        ![alt text](./img/Screen%20Shot%202022-07-04%20at%2019.40.20.png)


5. #### Create Folder Context and make an MyContext.cs File for Entity Framework can be build tables autoamte by the MyContex Script.

    * Before MyContext.cs Script creating in this Context folder, we're must have a few Nuget plugin :


        ![alt text](./img/Screen%20Shot%202022-07-04%20at%2022.12.41.png)
        ![alt text](./img/Screen%20Shot%202022-07-04%20at%2021.46.29.png)
        You can install they plugin in the Toold -> Nuget Package Solutions -> Browse, and they plugin it must still install on the old version (V. 5.0.11)
    * After Their plugin had install u can try create MyConetext.cs Files and typing this code in this File looks like this bellow:
         ![alt text](./img/Screen%20Shot%202022-07-04%20at%2022.18.02.png)

6. #### Configurations Database Connections and Startup.cs configurations.

    * Set appsetting.json file to be can connect with database MS. SQL Server by API variable configurations look like bellow of:
      ![alt text](./img/Screen%20Shot%202022-07-04%20at%2022.31.03.png)
    * After App setting Json was configure, you can try set MyContext as Context configurations sql server by Startup.cs files looks like bellow:
     ![alt text](./img/Screen%20Shot%202022-07-04%20at%2022.27.30.png)


7. #### type migration command in to NuGet Pacakage Console looks like bellow:

    ```
        add-migration 'lable migrations'
    ```

 ![alt text](./img/Screen%20Shot%202022-07-04%20at%2022.34.25.png)

 make sure migration success looks like that bellow:
  ![alt text](./img/Screen%20Shot%202022-07-04%20at%2022.37.20.png)

8. #### after type migration command we must typing 'database-update' to be implement entity database firsth code into real database our have:

        ```
        > update-database
        ```

  ![alt text](./img/Screen%20Shot%202022-07-04%20at%2022.39.32.png)

  make sure command succcss looks like that bellow:

  ![alt text](./img/Screen%20Shot%202022-07-04%20at%2022.42.12.png)

9. #### After this migration affected to databse we can show of the database have several tabel like what i write in the models as entity. 

 ![alt text](./img/Screen%20Shot%202022-07-07%20at%2018.40.35.png)

10. #### Migration Entity by Automate Framework has ben clear, next you can continue with CRUD API.

------
Crud Configurations
----------
11. #### Create Folder Repository on this Project 

12. #### Create Folder Interface on this Project
13. #### Create Interface File for Entity Data 
    * Create Interface File for Employee with Names "IRepository.cs".
    * TYpe on this files like thi bellow


    ````
        using System;
        using System.Collections.Generic;

        namespace APIKARYAWAN.Repository.Interface
        {
            public interface IRepository<Entity,Key> where Entity:class
            {
                IEnumerable<Entity> Get();
                Entity Get(Key key);
                int Insert(Entity entity);
                int Update(Entity entity);
                int Delete(Key key); 

            }
        }

    ````

14. #### Move back to Repository Folder and create GeneralRepository.cs type code likes bellow in this files :

````

using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using APIKARYAWAN.Context;
using APIKARYAWAN.Repository.Interface;
using Microsoft.EntityFrameworkCore;

namespace APIKARYAWAN.Repository // customize like your porject names
{
 public class GeneralRepository<Context,Entity, Key> : IRepository<Entity,Key>
  where Entity : class
  where Context : MyContext
 {
  private readonly MyContext myContext;
  private readonly DbSet<Entity> entities;
  public GeneralRepository(MyContext myContext)
  {
   this.myContext = myContext;
   this.entities = myContext.Set<Entity>();
  }
        public int Delete(Key key)
        {

            var entity = entities.Find(key);
            entities.Remove(entity);
            if (entity == null)
                throw new ArgumentNullException("entity");
            var respond = myContext.SaveChanges();
            return respond;

        }

        public IEnumerable<Entity> Get()
        {
            return entities.ToList();
        }

        public Entity Get(Key key)
        {
            return entities.Find(key);
        }

        public int Insert(Entity entity)
        {
            try
            {
                if (entity == null)
                    throw new ArgumentNullException("entity");
                entities.Add(entity);
                return myContext.SaveChanges();
            }
            catch (Exception)
            {
                return 0;
            }
        }

        public int Update(Entity entity)
        {
            if (entity == null)
                throw new ArgumentNullException("entity");
            myContext.Entry(entity).State = EntityState.Modified;
            return myContext.SaveChanges();
        }
    }
}



````

15. #### In the Repository Folder create Data Folder and Create Spesifc Repo for Entity and Type Code Likes Bellow:

```

using System;
using APIKARYAWAN.Context;

namespace APIKARYAWAN.Repository.Data
{
    public class EmployeeRepository :GeneralRepository<MyContext,Role,int>
    {
        public EmployeeRepository(MyContext myContext):base(myContext)
        {
        }
    }
}



```

16. #### Adds all repository data to the Startup files service like images bellow:


  ![alt text](./img/Screen%20Shot%202022-07-08%20at%2015.52.25.png)


17. #### Create Base Folder on this project and BaseController in this folder

 ![alt text](./img/Screen%20Shot%202022-07-08%20at%2016.45.36.png)

18. #### Create Base Folder on this project and BaseController in this folder

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using APIKARYAWAN.Repository.Interface;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

// For more information on enabling MVC for empty projects, visit https://go.microsoft.com/fwlink/?LinkID=397860

namespace APIKARYAWAN.Base
{
    [Route("api/[controller]")]
    [ApiController]
    public class BaseController<Entity, Repository, Key> : ControllerBase
        where Entity : class
        where Repository : IRepository<Entity, Key>
    {
        private readonly Repository repository;

        public BaseController(Repository repository)
        {
            this.repository = repository;
        }
        [HttpGet]
        public virtual ActionResult<Entity> Get()
        {
            try
            {
                var result = repository.Get();
                if (result.Count() > 0)
                {
                    // return Ok(result);
                    return Ok(new { status = StatusCodes.Status200OK, result, message = $" {result.Count()} data Found" });
                }
                else {
                    return BadRequest(new { status = StatusCodes.Status204NoContent, result, message = $"  [{ControllerContext.ActionDescriptor.ControllerName}] didn't have data" });
                }
            }
            catch (Exception e)
            {
                return BadRequest(new { status = StatusCodes.Status417ExpectationFailed, message = e.Message });
            }
        }
    
    [HttpGet("{Key}")]
    public ActionResult<Entity> Get(Key key)
    {
        try
        {
            var result = repository.Get(key);
            if (result != null)
            {
                return Ok(result);
                // return Ok(new { status = StatusCodes.Status200OK, result, message = $" Data Berhasil Didapatkan dengan parameter {key}" });
            }
            else
            {
                return Ok(result);
                //return BadRequest(new { status = StatusCodes.Status204NoContent, result, message = $"tidak ada indikasi data ditemukan di [{ControllerContext.ActionDescriptor.ControllerName}] dengan paramter {key}" });
            }
        }
        catch (Exception e)
        {
            return BadRequest(new { status = StatusCodes.Status417ExpectationFailed, errorMessage = e.Message });

        }
    }
    [HttpPost]
    public ActionResult<Entity> Post(Entity entity)
    {
        try
        {
            var result = repository.Insert(entity);
            switch (result)
            {
                case 1:
                    return Ok(result);
                // return Ok(new { status = StatusCodes.Status201Created, result, message = $"Data Berhasil Tersimpan ke [{ControllerContext.ActionDescriptor.ControllerName}]" });
                default:
                    return Ok(result);
                    //return BadRequest(new { status = StatusCodes.Status400BadRequest, result, message = $" Data gagal Ditambahkan Sudah ada di dalam database [{ControllerContext.ActionDescriptor.ControllerName}]" });
            }
        }
        catch (Exception e)
        {
            return BadRequest(new { status = StatusCodes.Status417ExpectationFailed, errors = e.Message });
        }
    }
    [HttpPut]
    public virtual ActionResult<Entity> Put(Entity entity)
    {
        try
        {
            var result = repository.Update(entity);
            switch (result)
            {
                case 1:
                    return Ok(result);

                // return Ok(new { status = StatusCodes.Status200OK, result, message = $"Data Berhasil Diubah dan Tersimpan ke [{ControllerContext.ActionDescriptor.ControllerName}]" });
                default:
                    return Ok(result);
                    // return BadRequest(new { status = StatusCodes.Status400BadRequest, result, message = $" Data gagal diubah di[{ControllerContext.ActionDescriptor.ControllerName}]" });
            }
        }
        catch (Exception e)
        {
            return BadRequest(new { status = StatusCodes.Status417ExpectationFailed, errors = e.Message });
        }

    }
    [HttpDelete("{key}")]
    public ActionResult<Entity> Delete(Key key)
    {
        try
        {
            var result = repository.Delete(key);
            switch (result)
            {
                case 1:
                    return Ok(result);
                //return Ok(new { status = StatusCodes.Status200OK, result, message = "Data Berhasil Dihapus" });
                default:
                    return Ok(result);
                    //return BadRequest(new { status = StatusCodes.Status400BadRequest, result, message = $" Data {key} Tidak ditemukan  atau sudah dihapus di[{ControllerContext.ActionDescriptor.ControllerName}]" });
            }
        }
        catch (Exception e)
        {
            return BadRequest(new { status = StatusCodes.Status417ExpectationFailed, errors = e.Message });
        }

    }

}
}


```

19. #### Create Controller for each Entitiy in this Controller Folder Like This:

 ![alt text](./img/Screen%20Shot%202022-07-11%20at%2009.37.37.png)

type this code simillar like this scritp bellow of:
```


using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using APIKARYAWAN.Base;
using APIKARYAWAN.Models;
using APIKARYAWAN.Repository.Data;
using Microsoft.AspNetCore.Mvc;

// For more information on enabling MVC for empty projects, visit https://go.microsoft.com/fwlink/?LinkID=397860

namespace APIKARYAWAN.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class EmployeesController : BaseController<Employees,EmployeeRepository,int>
    {
        private readonly EmployeeRepository employeeRepository;

        public EmployeesController(EmployeeRepository repository) : base(repository)
        {
            this.employeeRepository = repository;
        }
    }
}



```

20. #### Create Controller for each Entitiy in this Controller Folder Like This bellow:

 ![alt text](./img/Screen%20Shot%202022-07-11%20at%2016.22.05.png)


type this code simillar like this scritp bellow of:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using APIKARYAWAN.Base;
using APIKARYAWAN.Models;
using APIKARYAWAN.Repository.Data;
using Microsoft.AspNetCore.Mvc;

// For more information on enabling Web API for empty projects, visit https://go.microsoft.com/fwlink/?LinkID=397860

namespace APIKARYAWAN.Controllers
{
    [Route("api/[controller]")]
    [ApiController]

    public class BranchController : BaseController<Branch, BranchRepository, int>
    {
        private readonly BranchRepository branchRepository;
        public BranchController(BranchRepository repository) : base(repository)
        {
            this.branchRepository = repository;
        }
    }

}


```

21. #### Reconfigurations Startup.cs file and add all repository data in the scop services like this bellow 

 ![alt text](./img/Screen%20Shot%202022-07-11%20at%2016.23.42.png)

22. #### Build your the project by the klik Build in the Build Toolbar of Visual Studio

 ![alt text](./img/Screen%20Shot%202022-07-11%20at%2016.26.14.png)

23. #### After Buil Progress has successed, you can try Run this project by th klik 'Start Without Debuging or with Debuging' and try in Postman by they url:

 ![alt text](./img/Screen%20Shot%202022-07-11%20at%2016.27.49.png)

  ![alt text](./img/Screen%20Shot%202022-07-11%20at%2016.29.06.png)

  ![alt text](./img/Screen%20Shot%202022-07-11%20at%2016.29.27.png)

  ![alt text](./img/Screen%20Shot%202022-07-11%20at%2016.29.46.png)
    ![alt text](./img/Screen%20Shot%202022-07-11%20at%2016.29.46.png)

----
Custome Query in the Repository
----

23. #### create custome Query like searching query in the Repository with this step:
    * 
