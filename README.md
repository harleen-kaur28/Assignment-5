/*********************************************************************************
* WEB700 – Assignment 05
*
* I declare that this assignment is my own work in accordance with Seneca Academic Policy. No part
* of this assignment has been copied manually or electronically from any other source
* (including 3rd party web sites) or distributed to other students.
*
* Name:HARLEEN KAUR   Student_ID: 154157234  Date: 25/07/24
*
* Online (vercel) Link: 
*
********************************************************************************/

var HTTP_PORT = process.env.PORT || 8080;
var express = require("express");
const exphbs = require('express-handlebars');
const path = require('path');
var app = express();

app.use(express.urlencoded({ extended: true }));

app.use(express.static('public'));

app.engine('.hbs', exphbs.engine({ 
    extname: '.hbs',
    helpers: {
        navLink: function(url, options){
            return '<li' +
            ((url == app.locals.activeRoute) ? ' class="nav-item active" ' : ' class="nav-item" ') +
            '><a class="nav-link" href="' + url + '">' + options.fn(this) + '</a></li>';
        },
        equal: function (lvalue, rvalue, options) {
            if (arguments.length < 3)
            throw new Error("Handlebars Helper equal needs 2 parameters");
            if (lvalue != rvalue) {
            return options.inverse(this);
            } else {
            return options.fn(this);
            }
        }
    }
}));

app.set('view engine', '.hbs');

const collegeData = require('./modules/collegeData');

app.use(function(req,res,next){
    let route = req.path.substring(1);
    app.locals.activeRoute = "/" + (isNaN(route.split('/')[1]) ? route.replace(/\/(?!.*)/, "") : route.replace(/\/(.*)/, ""));
    next();
   });


app.get("/", (req, res) => {
    res.render('home');
});

app.get("/about", (req, res) => {
    res.render('about');
});

app.get("/htmlDemo", (req, res) => {
    res.render('htmlDemo');
});

app.get('/students/add', (req, res) => {
    res.sendFile(path.join(__dirname, 'views', 'addStudent.html'));
});

app.post('/students/add', (req, res) => {
    console.log(req.body);
    collegeData.addStudent(req.body).then(() => {
        res.redirect('/students');
    }).catch(err => {
        res.status(500).send("Unable to add student");
    });
});

app.get("/students", (req, res) => {
    if (req.query.course) {
        collegeData.getStudentsByCourse(req.query.course)
            .then((students) => {
                res.json(students);
            })
            .catch((err) => {
                res.json({ message: err });
            });
    } else {
        collegeData.getAllStudents()
            .then((data) => {
                res.render("students", {students: data}); 
            })
            .catch((err) => {
                res.render("students", {message: "no results"});
            });
    }
});


app.get('/student/:num', (req, res) => {
    collegeData.getStudentsByNum(req.params.num)
        .then(data => {
            res.render("student", {student: data[0]});
        })
        .catch(err => {
            res.render("student", {message: "no results"});
        });
});

app.post('/student/update', (req, res) => {
    let studentNum = parseInt(req.body.studentNum, 10);
    let course = parseInt(req.body.course, 10);

    if (isNaN(studentNum) || isNaN(course)) {
        return res.status(400).send('Invalid input: studentNum and course must be numbers');
    }

    const updatedStudent = {
        studentNum: studentNum,
        firstName: req.body.firstName,
        lastName: req.body.lastName,
        email: req.body.email,
        addressStreet: req.body.addressStreet,
        addressCity: req.body.addressCity,
        addressProvince: req.body.addressProvince,
        TA: req.body.TA === 'on',
        status: req.body.status,
    };

    collegeData.updateStudent(updatedStudent)
        .then(() => {
            res.redirect('/students');
        })
        .catch(err => {
            res.status(500).send("Unable to update student: " + err);
        });
});

app.get('/courses', (req, res) => {
    collegeData.getCourses().then(data => {
        res.render("courses", {courses: data});
    }).catch(err => {
        res.render("courses", {message: "no results"});
    });
});

app.get('/course/:id', (req, res) => {
    collegeData.getCourseById(req.params.id)
        .then(data => {
            res.render("course", {course: data[0]});
        })
        .catch(err => {
            res.render("course",{ message: "no results" });
        });
});

collegeData.initialize()
    .then(() => {
        app.listen(HTTP_PORT, ()=> {console.log("Server listening on port "+ HTTP_PORT)});
    })
    .catch((err) => {
        console.error(err);
    });
