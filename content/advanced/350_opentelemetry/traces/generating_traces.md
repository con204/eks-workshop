---
title: "Generating Traces"
date: 2021-5-01T09:00:00-00:00
weight: 52
draft: false
---

By default microservice demo is not configured to send tracing data to the ADOT collector. Run the following
commands to enable it:
```bash
sed -i 's/#//' kubernetes/backend/*.yaml
```

Then we apply these changes to our deployments
```bash
kubectl apply -f kubernetes/backend/
```

#### Adding a new Employee

Now that traces are enabled, let’s head back to our application and add a new employee. Let’s get our front-end URL:
```bash
echo http://${SERVICE_IP}/
```

Open that URL in your web browser, and create a new employee. You can use any First Name and Last Name. For Ocuppation choose one from the following table:

| Occupation |
|------------|
| Apprentice Software Engineer |
| Software Engineer |
| Senior Software Engineer |
| Lead Software Engineer |
| Principal Software Engineer |

For our example we will use "George Burdell" who's occupation is "Software Engineer".
![Screenshot of employee creation](/images/observability-with-adot/frontend-create-employee.png)

After clicking the 'ADD EMPLOYEE' button, you should get a success model with the Employee ID.
Let's copy that employee Id.
![Screenshot of employee creation success](/images/observability-with-adot/frontend-create-employee-success.png)

Then we'll close the modal, and copy that Employee ID into the Get Employee field.
![Screenshot of employee lookup](/images/observability-with-adot/frontend-lookup-employee.png)

After clicking 'GET EMPLOYEE' we'll get a modal with the Name & Occupation we provided earlier.
![Screenshot of employee lookup success](/images/observability-with-adot/frontend-lookup-employee-success.png)

