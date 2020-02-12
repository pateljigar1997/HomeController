# HomeController

using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading.Tasks;
using DEMO.Models;
using DEMO.ViewModels;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace DEMO.Controllers
{

    public class HomeController : Controller
    {

        private readonly IEmployeeRepository _employeeRepository;
        private readonly IHostingEnvironment hostingEnvironment;

        public HomeController(IEmployeeRepository employeeRepository,
                              IHostingEnvironment hostingEnvironment)
        {
            _employeeRepository = employeeRepository;
            this.hostingEnvironment = hostingEnvironment;

        }

        public ViewResult Index()
        {
           
               var model = _employeeRepository.GetAllEmployee();
            return View(model);
        }


        //public ActionResult Product()
        //{

        //    var model = _employeeRepository.GetAllEmployee();
        //    //IEnumerable<Product> lstProduct = new 
        //    List<Product> lstProduct = new List<Product>();
        //    Product p = new Product();
        //    p.Id = 1;
        //    p.Name = "TV";
        //    p.Email = "p1@yahoo.com";
        //    lstProduct.Add(p);
        //    //lstProduct.Append(p);
        //    Product p2 = new Product();
        //    p2.Id = 2;
        //    p2.Name = "Fridge";
        //    p2.Email = "p2@yahoo.com";
        //    lstProduct.Add(p2);

        //    return View(lstProduct.ToAsyncEnumerable());
        //}

        public ViewResult Details(int? Id)
        {
            //    throw new Exception("Error in Details View");
            Employee employee = _employeeRepository.GetEmployee(Id.Value);
            if (employee == null)
            {
                Response.StatusCode = 404;
                return View("EmployeeNotFound", Id.Value);
            }
            HomeDetailsViewModels homedetailsviewmodels = new HomeDetailsViewModels()
            {
                Employee = employee,
                Title = "details of employee"

            };
            return View(homedetailsviewmodels);

        }

        [HttpGet]
        public ViewResult Create()
        {
            return View();
        }


        [HttpGet]
        public ViewResult Edit(int 
            id)
        {
            Employee employee = _employeeRepository.GetEmployee(id);
            EmployeeEditViewModel employeeEditViewModel = new EmployeeEditViewModel
            {
                Id = employee.Id,
                Name = employee.Name,
                Email = employee.Email,
                Department = employee.Department,
                ExistingPhotoPath = employee.PhotoPath
            };
            return View(employeeEditViewModel);
        }


        [HttpPost]
        public IActionResult Edit(EmployeeEditViewModel model)
        {
            if (ModelState.IsValid)
            {
                Employee employee = _employeeRepository.GetEmployee(model.Id);
                employee.Name = model.Name;
                employee.Department = model.Department;
                employee.Email = model.Email;
                if (model.Photos != null)
                {
                    if (model.ExistingPhotoPath != null)
                    {
                        string filePath = Path.Combine(hostingEnvironment.WebRootPath, "Image", model.ExistingPhotoPath);
                        System.IO.File.Delete(filePath);
                    }
                    employee.PhotoPath = ProcessUploadedFile(model);
                }

                _employeeRepository.Update(employee);
                return RedirectToAction("index");
            }
            return View();
        }

        private string ProcessUploadedFile(EmployeeCreateViewModel model)
        {
            string uniqueFileName = null;
            if (model.Photos != null && model.Photos.Count > 0)
            {
                foreach (IFormFile photo in model.Photos)
                {
                    string uploadsFolder = Path.Combine(hostingEnvironment.WebRootPath, "Image");
                    uniqueFileName = Guid.NewGuid().ToString() + "_" + photo.FileName;
                    string filePath = Path.Combine(uploadsFolder, uniqueFileName);
                    using (var fileStream = new FileStream(filePath, FileMode.Create))
                    {
                        photo.CopyTo(fileStream);
                    }

                }
            }

            return uniqueFileName;
        }


        [HttpPost]
        public IActionResult Create(EmployeeCreateViewModel model)
        {
            if (ModelState.IsValid)
            {
                string uniqueFileName = ProcessUploadedFile(model);
                Employee newEmployee = new Employee
                {
                    Name = model.Name,
                    Email = model.Email,
                    Department = model.Department,
                    PhotoPath = uniqueFileName
                };
                _employeeRepository.Add(newEmployee);
                return RedirectToAction("Details", new { id = newEmployee.Id });
            }
            return View();
        }

        [HttpGet]
        public ActionResult Delete1(int id)
        {
            var create = _employeeRepository.GetEmployee(id);
            return View(create);
        }

        [HttpPost]
        [ActionName("Delete1")]
        public IActionResult Delete(int id)
        {
            _employeeRepository.Delete(id);
            _employeeRepository.Save();
            return RedirectToAction("index", "home");
        }
    }
}
