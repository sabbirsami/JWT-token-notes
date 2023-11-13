
# Sign Token using cookie
## Client site --
**I am using Axios to make POST requests to my server for handling JWT for (/jwt) and logout (/logout).**

**When a user `login`.**
```JS 
    axios
        .post(
            `https://server.vercel.app/jwt`,
            { email: currentUserEmail }, // Assuming the server expects an object with an 'email' property
            {
                withCredentials: true,
            }
        )
        .then((res) => {
            console.log("JWT creation response:", res.data);
        })
        .catch((error) => {
            console.error("Error creating JWT:", error);
        });
```
**When a user `logout`.**
```JS 
    axios
        .post(
            "https://server.vercel.app/logout",
            { email: currentUserEmail }, // Assuming the server expects an object with an 'email' property
            {
                withCredentials: true,
            }
        )
        .then((res) => {
            console.log("Logout response:", res.data);
        })
        .catch((error) => {
            console.error("Error logging out:", error);
        });

```
## Server site --
**Make sure you install jsonwebtoken in your server site and require it.**
```JS
const jwt = require("jsonwebtoken");
const cookieParser = require('cookie-parser');
```
```JS
app.use(cookieParser());
```
**Error Handling in `verifyToken` Middleware:**  </br>
 <small>You can handle errors from jwt.verify, but in case of an error, you send a response and then call next(). You need to consider returning after sending the response to avoid calling next() in case of an error.<small>
```JS
const verifyToken = (req, res, next) => {
    const token = req?.cookies?.token;
    if (!token) {
        return res.status(401).send({ message: "unauthorized access" });
    }
    jwt.verify(token, process.env.TOKEN_SECRET, (err, decoded) => {
        if (err) {
            return res.status(401).send({ message: "unauthorized access" });
        } else {
            req.user = decoded;
            next();
        }
    });
};

```
**`JWT route (/jwt)`**  </br>
The code you provided is a route handler for creating and sending a JWT as a cookie when a user logs in.

```JS 
app.post("/jwt", async (req, res) => {
    try {
        // Validate and sanitize user data here if needed

        const user = req.body;

        // Sign the JWT with user information
        const token = jwt.sign(user, process.env.TOKEN_SECRET, {
            expiresIn: "2h",
        });

        // Set the cookie with the JWT
        res.cookie("token", token, {
            httpOnly: true,
            secure: process.env.NODE_ENV === "production",
            sameSite: process.env.NODE_ENV === "production" ? "none" : "strict",
            // Adjust the cookie expiration time if needed
            maxAge: 2 * 60 * 60 * 1000, // 2 hours in milliseconds
        }).send({ success: true });
    } catch (error) {
        console.error("Error creating JWT:", error);
        res.status(500).send({ success: false, message: "Internal Server Error" });
    }
});

```
**`Logout Route (/logout)`**  </br>
```JS 
app.post("/logout", async (req, res) => {
    const user = req.body;
    console.log("logging out", user);
    res.clearCookie("token", {
        httpOnly: true,
        secure: process.env.NODE_ENV === "production",
        sameSite: process.env.NODE_ENV === "production" ? "none" : "strict",
    }).send({ success: true });
});

```
**`SameSite Attribute for Cookies:`**  </br>
In your /jwt route, if you're setting sameSite: "none" for the cookie, which is suitable for cross-site requests (e.g., when your frontend is hosted on a different domain). Make sure that your production environment uses HTTPS because the "none" value for sameSite requires a secure connection.
```JS 
sameSite: process.env.NODE_ENV === "production" ? "none" : "strict",
```

</hr>



# Sign Token using localhost
## Client site --
The code begins with a conditional statement checking if the variable email has a truthy value. In JavaScript, a truthy value is any value that is not explicitly falsy (e.g., `null, undefined, false, 0, an empty string, or NaN`). If the email is truthy, the block of code inside the if statement will be executed.
```JS 
 if (email) {
    fetch(`https://server.vercel.app/user/${email}`, {
      method: "PUT",
      headers: {
            "content-type": "application/json",
                },
            body: JSON.stringify(currentUser),
            })
            .then((res) => res.json())
            .then((data) => {
             const accessToken = data.token;
             console.log(accessToken);
             localStorage.setItem("accessToken", accessToken);
             setToken(accessToken);
        });
}
```
## Server site --
```JS 
 function verifyJWT(req, res, next) {
     const authHeader = req.headers.authorization;
     if (!authHeader) {
         return res.status(401).send({ message: "Unauthorized access" });
     }
     const token = authHeader.split(" ")[1];
     jwt.verify(token, process.env.WEB_TOKEN, function (err, decoded) {
         if (err) {
            return res.status(403).send({ message: "Forbidden access" });
         }
         req.user = decoded;
         next();
     });
 }
```
