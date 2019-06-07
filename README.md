# Live-Project
Throughout the final phase of my education at The Tech Academy, I worked on a team project developing a full-scale MVC web application in C#. Throughout the process, I collaborated with my peers on the legacy code to fix bugs, streamline code and build new tools for the customer. Through observing my peers and project manager, I learned about how to communicate successes and complications. I learned how to ask questions and think like a developer. Because there were students solely committed to front-end, I only had the opportunity to work on back-end stories. I'm proud of the work that my team and I accomplished. Many of us were remote, as well, but we managed to create a playful and productive team dynamic. Most of all, the live project helped me to build confidence in my programming skills and my ability to learn quickly and independently. 

Below are descriptions of the stories I worked on, along with snippits of code.

<h2>Back End Stories</h2>
<ul>
<li><a href="#redirect">Redirect to Dashboard</a></li>
<li><a href="#fix-return-to-dashboard"> Fix Returns to Dashboard</a></li>
<li><a href="#company-news-partial">Fix Company News Partial Links</a></li>
</ul>
<h3><a id="redirect" class="anchor" aria-hidden="true" href="#redirect"><svg class="octicon octicon-link" viewBox="0 0 16 16" version="1.1" width="16" height="16" aria-hidden="true"><path fill-rule="evenodd" d="M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z"></path></svg></a>Redirect to Dashboard</h3>

If the site has saved your log in information so that you are logged in when you first visit the site, currently you'll still end up on the landing page. My job was to set the landing page to automatically redirect logged in users to the Dashboard.
<pre><code> //Redirect to dashboard when user log in info is remembered by browser

using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace ConstructionNew.Controllers
{
    public class HomeController : Controller
    {
        // GET: Home
        public ActionResult Index()
        {
            if (User.Identity.IsAuthenticated)
            {
                return Redirect("/Dashboard");
            }
            else
            {
                return View();
            }
        }
     }
}     
</code></pre>

<h3><a id="fix-return-to-dashboard" class="anchor" aria-hidden="true" href="#fix-return-to-dashboard"><svg class="octicon octicon-link" viewBox="0 0 16 16" version="1.1" width="16" height="16" aria-hidden="true"><path fill-rule="evenodd" d="M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z"></path></svg></a>Fix Return To Dashboard</h3>

Once the team added a new landing page, an unintended consequence of moving the dashboard was that all the return actions that went to the dashboard no longer worked. My job was to fix these links to return to the dashboard index, not the home index. I then checked all the partial views to ensure that on refresh or return, they loaded the dashboard instead of the home index.

<code><pre>
[HttpPost]
[ValidateAntiForgeryToken]
[Authorize(Roles = "Admin")]
        public ActionResult Create([Bind(Include = "Title,NewsItem,ExpirationDate")] CompanyNews companyNews)
        {
            if (ModelState.IsValid)
            {
                companyNews.DateStamp = DateTime.Now.Ticks.ToString();
                db.CompanyNews.Add(companyNews);
                db.SaveChanges();
                return RedirectToAction("Index", "Dashboard");
            }

            return View(companyNews);
        }
 [HttpPost]
 [AllowAnonymous]
 [ValidateAntiForgeryToken]
        public async Task<ActionResult> Register(RegisterViewModel model)
        {
            if (ModelState.IsValid)
            {
                var user = new ApplicationUser
                {
                    UserName = model.UserName,
                    Email = model.Email,
                    PhoneNumber = model.PhoneNumber,
                    FName = model.FName,
                    LName = model.LName,
                    // Get user role
                    UserRole = model.UserRoles
                };
                var identityResult = await UserManager.CreateAsync(user, model.Password);
                if (identityResult.Succeeded)
                {
                    // assign user to role
                    await UserManager.AddToRoleAsync(user.Id, user.UserRole);

                    await SignInManager.SignInAsync(user, false, false); //#1 comment this line of code to test email confirmation link                     below #2

                    // For more information on how to enable account confirmation and password reset please visit                                           https://go.microsoft.com/fwlink/?LinkID=320771
                    // Send an email with this link
                    //#2 uncomment lines of code below to test email confirmation link
                    //string code = await UserManager.GenerateEmailConfirmationTokenAsync(user.Id);
                    //var callbackUrl = Url.Action("ConfirmEmail", "Account", new { userId = user.Id, code = code }, protocol:                               Request.Url.Scheme);
                    //await UserManager.SendEmailAsync(user.Id, "Confirm your account", "Please confirm your account by clicking <a                         href=\"" + callbackUrl + "\">here</a>");

                    // Delete CreateUserRequest
                    using (db)
                    {
                        var createUserRequest = db.CreateUserRequests.First(x => x.UserName == user.UserName);
                        if (createUserRequest != null) db.CreateUserRequests.Remove(createUserRequest);
                        db.SaveChanges();
                    }

                    return RedirectToAction("Index", "Dashboard");
                }
            }
            // If we got this far, something failed, redisplay form
            return View(model);
        }
@*Partial view - return to dashboard*@
@model ConstructionNew.Models.CompanyNews
@{
    ViewBag.Title = "Create";
}

@using (Html.BeginForm("Create", "CompanyNews", FormMethod.Post))

{
    @Html.AntiForgeryToken()

    <form id="class_Form">

        <!-- Button trigger modal -->
        <br />
        <button type="button" class="btn btn-primary btn-lg" style="color:#FFF; font-weight:600" data-toggle="modal" data-target="#myModal">
            Create Company News
        </button>

        <!-- Modal -->

        <div class="form-group  modal-news">
            @Html.LabelFor(model => model.Title, htmlAttributes: new { @class = "control-label-lg" })
            <br />
            @Html.TextBoxFor(model => model.Title, new { htmlAttributes = new { @class = "form-control", @placeholder = "Title" } })
            @Html.ValidationMessageFor(model => model.NewsItem, "", new { @class = "text-danger" })
        </div>
        <div class="form-group  modal-news">
            @Html.LabelFor(model => model.NewsItem, htmlAttributes: new { @class = "control-label" })
            <br />
            @Html.TextAreaFor(model => model.NewsItem, new { htmlAttributes = new { @class = "form-control", @placeholder = "News" } })
            @Html.ValidationMessageFor(model => model.NewsItem, "", new { @class = "text-danger" })
        </div>
        <div class="form-group modal-news">
            @Html.LabelFor(model => model.ExpirationDate, htmlAttributes: new { @class = "date" })
            <br />
            @Html.EditorFor(model => model.ExpirationDate, new { htmlAttributes = new { @class = "datepicker", type = "date" } })
            @Html.ValidationMessageFor(model => model.ExpirationDate, "", new { @class = "text-danger" })
        </div>
        <div class="modal-footer">
            <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
            <button type="submit" class="btn btn-primary">Save changes</button>
        </div>
        <div>
            @Html.ActionLink("Back to List", "Index") |
            @Html.ActionLink("Back to Dashboard", "Index", "Dashboard")
        </div>

    </form>
</code></pre>

<h3><a id="company-news-partial" class="anchor" aria-hidden="true" href="#company-news-partial"><svg class="octicon octicon-link" viewBox="0 0 16 16" version="1.1" width="16" height="16" aria-hidden="true"><path fill-rule="evenodd" d="M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z"></path></svg></a>Fix Company News Partial Links</h3>

The links in the Company News partial view were not set up properly to access the views they are supposed to refer to. My job was to modify these links so they worked correctly. It was also necessary to restrict Edit and Delete links to Admin only. Additionally, the pop-up for Create News needed to be redirected to return the dashboard upon submit. There were some additional problems with the legacy code that had to be solved in order to make the preceeding tasks work, including migrated data in an incorrect format and the date formatting not matching in views and controllers. See file in repo for more of this code. 

<code><pre>
//return Edit/Delete/Details Action Links to the company news index

        if (item.ExpirationDate.GetValueOrDefault(DateTime.Now).Ticks >= DateTime.Now.Ticks)
       
        @Html.ActionLink("Edit", "Edit", new { id = item.DateStamp }, "CompanyNews") |
        @Html.ActionLink("Delete", "Delete", new { id = item.DateStamp }, "CompanyNews") |
        @Html.ActionLink("Details", "Details", new { id = item.DateStamp }, "CompanyNews") |

//------------------------------------------------------------------------------------------------------
//Edit the model so that date formats match with controller

namespace ConstructionNew.Models
{
    public class CompanyNews
    {
        [Key]
        [Display(Name = "Date")]
        [DisplayFormat(DataFormatString = "{0:MM/dd/yy hh:mm tt}")]
        public string DateStamp { get; set; }
        [Display(Name = "Title")]
        public string Title { get; set; }
        [Display(Name = "Body")]
        public string NewsItem { get; set; }
        [DisplayFormat(DataFormatString = "{0:MM/dd/yy hh:mm tt}")]
        public Nullable<DateTime> ExpirationDate { get; set; }
}
}

//-------------------------------------------------------------
// Edit Controller so that submit links return to company news index

namespace ConstructionNew.Controllers
{
   
    public class CompanyNewsController : Controller
    {
        private ApplicationDbContext db = new ApplicationDbContext();

        // GET: CompanyNews
        [Authorize] //used to restrict view of unregistered users
        public ActionResult Index()
        {
            return View(db.CompanyNews.ToList());
        }

        // Testing for work item 4341: Display Company News items with 
        // Expiration dates that have not yet occurred, or are NULL.
        public ActionResult UnexpiredNews()
        {
            var data = db.CompanyNews
                .Where(x => !x.ExpirationDate.HasValue
                || DbFunctions.DiffHours(x.ExpirationDate, DateTime.Now) <= 0);

            return View(data.ToList());
        }

        // GET: CompanyNews/Details/5
        [Authorize] //used to restrict view of unregistered users
        public ActionResult Details(string id)
        {
            if (id == null)
            {
                return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
            }
            CompanyNews companyNews = db.CompanyNews.Find(id);
            if (companyNews == null)
            {
                return HttpNotFound();
            }
            return View(companyNews);
        }

        // GET: CompanyNews/Create
        [Authorize(Roles = "Admin")]
        public ActionResult Create()
        {
            return View();
        }

        // POST: CompanyNews/Create
        // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
        // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        [Authorize(Roles = "Admin")]
        public ActionResult Create([Bind(Include = "Title,NewsItem,ExpirationDate")] CompanyNews companyNews)
        {
            if (ModelState.IsValid)
            {
                companyNews.DateStamp = DateTime.Now.Ticks.ToString();
                db.CompanyNews.Add(companyNews);
                db.SaveChanges();
                return RedirectToAction("Index", "Dashboard");
            }

            return View(companyNews);
        }

        // GET: CompanyNews/Edit/5
        [Authorize(Roles = "Admin")]
        public ActionResult Edit(string id)
        {
            if (id == null)
            {
                return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
            }
            CompanyNews companyNews = db.CompanyNews.Find(id);
            if (companyNews == null)
            {
                return HttpNotFound();
            }
            return View(companyNews);
        }

        // POST: CompanyNews/Edit/5
        // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
        // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        [Authorize(Roles = "Admin")]
        public ActionResult Edit([Bind(Include = "DateStamp,Title,NewsItem,ExpirationDate")] CompanyNews companyNews)
        {
            if (ModelState.IsValid)
            {
                db.Entry(companyNews).State = EntityState.Modified;
                db.SaveChanges();
                return RedirectToAction("Index");
            }
            return View(companyNews);
        }

        // GET: CompanyNews/Delete/5
        [Authorize(Roles = "Admin")]
        public ActionResult Delete(string id)
        {
            if (id == null)
            {
                return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
            }
            CompanyNews companyNews = db.CompanyNews.Find(id);
            if (companyNews == null)
            {
                return HttpNotFound();
            }
            return View(companyNews);
        }

        // POST: CompanyNews/Delete/5
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        [Authorize(Roles = "Admin")]
        public ActionResult DeleteConfirmed(string id)
        {
            CompanyNews companyNews = db.CompanyNews.Find(id);
            db.CompanyNews.Remove(companyNews);
            db.SaveChanges();
            return RedirectToAction("Index");
        }

        public ActionResult _CompanyNews()
        {
            return PartialView(db.CompanyNews.AsEnumerable());
        }

        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                db.Dispose();
            }
            base.Dispose(disposing);
        }

    }
}
</code></pre>
