> Did you attend online school this year?
> 
> Good, because you'll need to register at grading.hsc.tf and get an A on "simple quiz" to find the flag.
> 
> Server code is attached.

The provided webpage first asks for a login / register, after which two quizzes are shown. The "simple quiz" shows that the deadline has passed, whereas "another simple quiz" does not.

Inspecting the form view (`views/form.ejs`):

```ejs
...
<% if(form.submitted) { %>
    <h2>This form has been submitted.</h2>
<% } %> 
<% if(after.late) { %> 
    <h2>The deadline has passed. You got <%= after.grade %> questions right.</h2>
    <% if(after.grade === form.questions.length) { %> 
        <p>here is the flag: <b><%= after.flag %></b></p>
    <% } %> 
<% } else { %> 
    <p>You can edit until <%= form.deadline %>.</p>
    
<% } %> 
</div>
...
```

This shows that the flag is shown when the deadline has passed (note: the second question's deadline was *not* meant to pass during the ctf, but did anyway because of rescheduling) and the grade is equal to the number of questions.

Inspecting the server code (`app.js`):

```js
app.route("/register")
...
.post((req, res) => {
    User.register({username: req.body.username}, req.body.password, (err, user) => {
        if(err) {
            ...
        } else {
            const question1 = {
                question: "What is the capital of Africa?",
                choices: [
                    "Venezuela",
                    "Kalibloom",
                    "Nairobi",
                    "Tokyo",
                    "Africa is not a country"
                ],
                answer: "Africa is not a country",
                submission: "Kalibloom",
            }

            const question2 = {
                question: "What is the best CTF?",
                answer: "HSCTF",
            }

            user.questions.push(question1)
            user.questions.push(question2)
            user.save()

            console.log(user.questions);

            const failedTest = {
                name: "simple quiz",
                questions: [
                    user.questions[0]._id,
                ],
                deadline: new Date(2021, 5, 13, 0, 0, 0)
            }
            
            const newTest = {
                name: "another simple quiz",
                questions: [
                    user.questions[1]._id,
                ],
                deadline: new Date(2021, 5, 19, 0, 0, 0)
            }

            user.forms.push(failedTest)
            user.forms.push(newTest)
            user.save()

            passport.authenticate("local")(req, res, () => {
                res.redirect("/")
            })
        }
    })
})
```

This gives the answers to the questions (in case there was any doubt), and also shows that the questions and forms are handled separately. Inspecting further:

```js
app.route("/:formID")
.get(authMW, (req, res) => {
    const formID = req.params.formID
    const form = req.user.forms.id(formID)
    if(!form) {
        res.redirect("/") // not found
    } else {

        const payload = {
            name: form.name,
            deadline: form.deadline,
            questions: []
        }

        for(let q in form.questions) {
            form.questions[q] = req.user.questions.id(form.questions[q])
        }

        after = {}

        const late = Date.now() > form.deadline
        after.late = late

        if(late) {
            let grade = 0
            for(let q of form.questions) {
                if (q.submission == q.answer) grade += 1
            }

            after.grade = grade
        }

        after.flag = process.env.FLAG

        // console.log(form, form.questions[0]);
        res.render("form.ejs", {form: form, after: after})
    }
})
```

shows that the grade is computed directly from the matching answers. The `POST` for the same endpoint:

```js
.post(authMW, (req, res) => {
    const now = Date.now()
    const form = req.user.forms.id(req.params.formID)
    if(now > form.deadline) {
        res.json({response: "too late"})
    } else {
        if(req.body.ID) {
            const question = req.user.questions.id(req.body.ID)
            console.log(question);
            question.submission = req.body.value
            req.user.save()
        } else {
            form.submitted = true
            req.user.save()
        }

        res.json({response: "heh"})
    }

})
```

This shows the vulnerability: no submissions for a form whose deadline has passed will be accepted, but since the questions and forms are handled separately and there are no checks to tie the two, submitting a form allows one to change the answer for any question.

Using this, a `POST` request can be forged that uses the form id of the second form ("another simple quiz"), but the question id (obtained from the first form) and the correct answer for the first form's question. This changes the answer and the flag is printed on the first form's page: `flag{th3_an5w3r_w4s_HSCTF_0bvi0us1y}`.
