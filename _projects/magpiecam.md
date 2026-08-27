---
layout: project
title: "MagPieCam"
demo_video: "/assets/videos/magpiecam/demo.mp4"
demo_video_poster: "/assets/videos/magpiecam/demo-poster.jpg"
demo_video_caption: "Adding a camera by QR code, writing a rule in plain English, and receiving the notification it triggers."
tagline: 'A smart camera system that enables users to define the notifications they want in plain English using the <a href="https://deepmind.google/models/gemma/gemma-4/">Gemma4</a> / <a href="https://ai.google.dev/gemini-api/docs/models/gemini-3.1-flash-lite">Gemini 3.1 Flash-Lite</a> and <a href="https://docs.ultralytics.com/tasks/detect">Yolo</a>.'
image: "/assets/images/projects/magpie/magpicam.jpg"
technologies:
  - "Python"
  - "Django Rest Framework"
  - "ByteTrack"
  - "Yolo"
  - "Gemma 4"
  - "Gemini 3.1 Flash-Lite"
  - Pydantic
  - "PostgreSQL"
  - "Terraform"
  - "RESTful API"
  - "AWS (SQS, S3, Fargate)"
github_url: "https://github.com/ataffe/MagPieCam-Core"
api_docs_url: "/projects/magpiecam/api-docs/"
features:
  - "Users can add new cameras to their account seamlessly via a qr code"
  - "View the live video"
  - "Users can add rules using natural language for notifications"
  - "Each notifications includes a 20 second video clip that includes the 10 seconds leading up to the event."
ios_screenshots:
  - "/assets/images/projects/magpie/ios-qrcode.PNG"
  - "/assets/images/projects/magpie/ios-camera-detail.PNG"
  - "/assets/images/projects/magpie/ios-rules.PNG"
  - "/assets/images/projects/magpie/ios-notifications.PNG"
architecture_image: "https://raw.githubusercontent.com/ataffe/MagPieCam-Assets/main/system_diagram/magpie-cam-system-diagram.png"
architecture_features:
  ios app: 
    technologies:
    - "Swift"
    - "SwiftUI"
    features:
      - "The MagPiCam iOS app consumes the MagPieCam-Core's REST API enabling users can manage rules and notifications"
      - "Live video is streamed from the MediaMTX server using WebRTC"
  backend:
    technologies:
      - "Python"
      - "Django Rest Framework"
      - "Celery"
      - "Gemma 4"
      - "Gemini 3.1 Flash-Lite"
      - "PostgreSQL"
      - "Redis"
      - "AWS (SQS, S3, Fargate)"
    features:
      - "Django REST frameworks handles authentication and CRUD for users, cameras, rules, and notifications via a REST API secured with JWTs."
      - "Models are stored in a Postgres DB and use UUIDv7 ids as primary keys for sortable, opaque, index-friendly IDs"
      - "The event processor (a Django app) consumes SQS messages, and passes the payload to a Celery Worker"
      - "Celery workers evaluate images against user rules using the rules engine and triggers notifications."
      - "Cameras retrieve their own JWT based on a revokable device id created from the Raspberry Pi cpu serial number and a UUID"
      - "The core services and DB's run in AWS using Fargate, RDS PostgreSQL, and ElastiCache Redis"
  edge agent:
    technologies:
      - "C++"
      - "ByteTrack"
      - "Yolo"
    features:
      - "Tracks objects using ByteTrack+Yolo and sends images to the backend with a per track exponential backoff"
      - "Streams video when a user connects to the MediaMTX server, by long polling a REST endpoint."
      - "Presigned S3 URLs let edge cameras upload images & video clips directly to storage (S3)"
related_repos:
  - name: "MagPieCam-Core"
    url: "https://github.com/ataffe/MagPieCam-Core"
    description: "The Django backend and celery workers."
  - name: "MagPieCam-iOS"
    url: "https://github.com/ataffe/MagPieCam-iOS"
    description: "The iOS app"
  - name: "MagPieCam-EdgeAgent"
    url: "https://github.com/ataffe/MagPieCamEdgeAgent"
    description: "The Edge Agent that runs on the camera."
featured: true
intro: MagPieCam lets users choose what they want to be notified about using rules they create in natural language. The camera uses an object tracker (<a href="https://github.com/FoundationVision/ByteTrack">ByteTrack</a>) to recognize moving objects and then sends an images to the rules engine (Gemma4 / Gemini API), that decides if the image should trigger a push-notification. The camera itself is built using a Raspberry Pi Zero 2W and the Raspberry Pi AI Camera. On this page I will walk through the architecture of the system and some of the key parts of building it.

motivation: <strong>I wanted to build a system that is centered around an AI model, does something useful, and is built for production</strong>. Not just a dataset and a model with performance metrics but a full system in the cloud that works and scales. I also have two cats at home named Zia and Luna! They are 3 years old and very curious, so I need to keep an eye on them sometimes. Luna for example, has some health problems, and so sometimes I need to track when she is eating or using the bathroom. So I tried setting up a Ring cam near their litter box or food bowls for example. But <strong> I get many notifications for events that are not what I am looking for</strong>. For example, every time a cat or person walks by, I get an event. Additionally my girlfriend and I live in an area that has a decent amount of foot traffic. Similarly we tend to get a lot of notifications that are just people walking by. Ring now has unusual event detection but the user doesn't determine what is unusual. So I built MagPieCam.  

note1: I used <a href="https://code.claude.com/docs/en/overview">Claude Code</a> throughout this project. I wanted to build a scalable, reliable AI system end to end, but you can't be an expert in everything, so I focused on the AI Engineer, the REST API and AWS with Terraform, and delegated the majority of the iOS and Edge development to Claude code. I talk more about my workflow below.
image1: "/assets/images/projects/magpie/zia_luna.jpg"
claude_workflow:
  intro: When I started this project, I wanted to solve a problem and learn about AI Engineering by building a backend system that used an AI model as if it were in production. However, I wanted to build the backend in the context of a fully working system. If I built just the backend, it would be very hard to conceptualize some of the interesting problems that I encountered while connecting the iOS app to the edge device, like the streaming lifecycle for example. So I wrote most the MagPieCam-Core services myself and delegated the a lot of the iOS app development and the edge agent development to Claude Code. When designing the system, I would come up with a design myself and have Claude play systems architect and review the design for anything I was missing or help me understand how engineering teams typically solve the problem.
  practices:
    - title: "Backend by hand"
      description: "I usually started with a problem I want to solve, then would design a feature to solve it and then review it with Claude. When reviewing with Claude, I would usually ask clarifying questions and how engineering team at existing companies have solved the problem."
    - title: "Delegated iOS & Edge Agent"
      description: " When developing the iOS app, I would usually add a new REST endpoint, empty view, or view model and describe to Claude what I wanted to implement. Then either I would teak the view by hand or I would ask Claude tweak the design (mostly Claude) until it looked good. </br></br>When building the edge agent I deployed YOLOv11 to the Raspberry Pi AI Camera and implemented the tracking algorithm in Python. Then I had Claude port it to C++ and then I reviewed it and made tweaks. Then I went feature by feature with claude writing the code for features like the FFMPEG streamer."
  claude_image: "/assets/images/projects/magpie/claude-code-128px.png"
  takeaway: "I used to be very skeptical about delegating work to agents like Claude Code, but it really accelerated the development of this project. So far I have noticed that <strong>most mistakes come from miscommunications</strong>, so I try to be as specific as possible when working with Claude even if I don't know how to do what I want. Developing the app is a great case study of this. </br></br>I started by learning Swift using Stanford's <a href='https://cs193p.stanford.edu/'>cs193p course</a>. Then I started building the app. At first I built the views myself, but I am new to UI design and as views got more complex it was taking too much of my time. Swift has a vast library of tools for building views, so to save time I focused learning from other UIs so I could describe what I want and let Claude focus on how to implement that using best practices."
architecture:
  overview: "I think the most digestible way to look at the system is by looking at it feature by feature in the order that a user might encounter each feature. (Click on any image on the right to enlarge it)"
  section1:
    title: "Creating an Account"
    content: "User account are pretty straightforward and implemented using Django's User model with the simple JWT plugin. When a user creates an account, the app receives a short lived access token and a refresh token that can be used to retrieve a new access token. All other endpoints besides the device provision ones require JWTs."
    images:
      - src: "/assets/images/projects/magpie/ios-login.PNG"
        caption: "The login screen."
      - src: "/assets/images/projects/magpie/ios-create-account.PNG"
        caption: "Creating an account."
    image_columns: 2
  section2:
    title: "Adding a camera"
    subsections:
      - title: "Generating the Claim Token"
        content: >-
          First to create a camera in the system I create a QR code using a
          <code>claim_token</code> which is generated by the <code>cameras/provision</code>
          endpoint. The cameras/provision endpoint takes a device id (CPU serial
          number on the Raspberry Pi) and generates a claim_token by appending a
          uuid to the device id. The claim_token is then hashed using SHA-256,
          stored in a Camera model in Postgres along with the device id. Then the text token is
          returned to the caller.
      - title: "Why the Claim Token Is Single-Use"
        content: >-
          This process basically simulates what would usually take
          place when the cameras are assembled. The claim_token can only be used once
          to pair a camera to a user's app. This design ensures that if the camera is stolen it
          can't be claimed. But before the user can claim the camera it must register
          with the backend at start up.
      - title: "Registering the Camera"
        content: >-
          At startup, the camera calls the <code>cameras/register</code> endpoint passing it's device id and the claim_token.
          The backend core then queries the DB for the device id and compares the stored hashed claim_tokens to the hashed input claim_token. If a matching camera is found in the DB, a flag is set on the camera model and another uuid is generated and
          attached to the device id, creating the <code>device_token</code>. This
          token is saved to disk in the camera and is used to request JWTs going forward.
          This system enables the device token to be revoked in the case that the
          camera is lost or stolen.
      - title: "Claiming the Camera"
        content: >-
          Once the camera has been registered the app can scan the QR code which calls <code>cameras/claim</code> and sets the claimed.
    images:
      - src: "/assets/diagrams/magpie/DeviceProvisioningSeqDiagram.png"
        caption: "The device provisioning sequence</br>(click to see full view)"
      - src: "/assets/images/projects/magpie/ios-qrcode.PNG"
        caption: "Add a camera via QR code"
    image_columns: 2
  section3:
    title: "Detecting Moving Objects"
    content: "The edge agent on the camera kind of operates outside of what the user does. It tracks moving objects and uploads detection images regardless of what is going on. The camera is built using a Raspberry Pi Zero 2W and the Raspberry Pi AI Camera. The AI camera is running a quantized YOLOv11n model and passes the bounding boxes to the ByteTrack tracker.
    
    
    Each track has a it's own exponential backoff, so that you don't get bursts of images of the same thing uploaded to S3. I used this approach because an object may enter the view of the camera but it may take a while for it to do something that triggers a notification. For example my cat can walk into view but not eat for 30 seconds. Each time an object is detected the camera requests a short-lived (currently good for 5 minutes) presigned url from <code>/cameras/presigned_upload</code> in order to upload the image to S3. Using a presigned url enable the camera to upload images without permanently storing AWS credentials. The <code>image_detection</code> S3 bucket then enqueues a message in SQS on upload events. (I am going to write an article that goes into more depth on the camera soon)"

    images:
      - src: "https://raw.githubusercontent.com/ataffe/MagPieCam-Assets/main/system_diagram/ImageDetectionDiagram.png"
        caption: The image detection pipeline.
  section4:
    title: "Creating & Evaluating Rules"
    subsections:
      - title: "Adding Rules"
        content: >- 
          Once a camera has been added to a user's account they can add rules for smart notifications to the camera.
          In the iOS app a prefix is hardcoded to give the rules a predictable structure. As it is now, all rules need to be prefixed with <strong>'Tell me when you see:'</strong> and then the user fills in the rule. This and other constraints like structured output make the model's output more predictable. The rules model itself in Django is pretty straightforward: it stores the rule suffix, has it's own id <code>public_rule_id</code> that is a uuid7 like the other public ids, and has an field <code>enabled</code> for toggling the rule.
    
      - title: "Evaluating Detection Images"
        content: "The event processor picks up messages from SQS which have the S3 object key needed to download the image. The event processor then creates a task for Celery workers, and thats pretty much it for the event processor. It just runs in a loop consuming from SQS. One important thing it does <strong>NOT</strong> do is mark the message as delivered. That is done by the celery worker so that if something goes wrong (Gemini API is down etc.) the message can be delivered again. The celery worker then downloads the image from S3, builds a prompt from the user's rules and then runs inference either using the rules model."

      - title: "Prompting"
        content: "The full prompt is composed of: 
        </br>
        </br>
        <u>System Instructions</u>
        </br>
        </br>
        <code>
        SYSTEM_INSTRUCTIONS =
            'You evaluate whether a condition is present in a security camera image.
            Answer only from what is visibly present. If the image is too dark,
            blurry, or ambiguous to tell, answer 'unsure' rather than guessing.'
        </code>
        </br>
        </br>
        Currently I am using <a href='https://www.kaggle.com/models/google/gemma-4/transformers/gemma-4-e2b-it'>gemma4-e2b-it</a> as the hosted model in dev (more on this below) and I noticed that it tends to trigger false alerts in darker images. So the goal of the system prompt is to make the model lean toward <strong>false negatives over false positives</strong>. This idea here is, even though the instructions say the camera is a security camera, the use cases for the camera broader than security camera. For example going forward the plan is to add analytics so you can see how many events triggered for a rule and how often. So the idea is to focus on highly valuable notifications.
        </br>
        </br>
        <u>Rules Preamble</u>
        </br>
        </br>
        <code>
        RULES_PREAMBLE = 'For each condition below, answer whether it is visible in the image.'
        </code>
        </br>
        </br>
        <u>Rules list</u>
        </br>
        </br>
        The rules are then combined with the prefix <strong>'Is there'</strong> because as seen above the format of the rules in the database is:
        </br>
        </br>
        * a person eating
        </br>
        * a blue car in the driveway
        </br>
        * a cat on the counter
        </br>
        </br>
        Each rule is then listed with it's <code>public_rule_id</code>, so the full rules list would look like:
        </br>
        </br>
        <code>'01a01b0f-2cf3-7110-8f89-9c59f02d2c51:Is there a person eating.\n
        01a01b0f-2cf3-7110-8f89-9c59f02d2c52:Is there a car in the driveway.\n
        01a01b0f-2cf3-7110-8f89-9c59f02d2c53:Is there a cat on the counter.'</code>


        Structuring the rules like this enables the model to evaluate all the rules in a single prompt or inference run. This reduces the number of api calls made to the Gemini API in production. It also means the system instructions, preamble are only included once as opposed to multiple times if calling the API for each rule, which means <strong>less tokens, and less money spent</strong>. When using the Gemini API the prompt end here because it can force the model to adhere to a json schema using structured outputs. So I created a pydantic model for the output and pass that to the Gemini API. When hosting the model myself I am currently using a json response prompt along with some post-processing like removing a json code fence (```json) for example:
        </br>
        </br>
        <code>
        JSON_RESPONSE_INSTRUCTIONS =

            'Reply with JSON and nothing else. No prose, no explanation, no code fences.
            Use exactly this shape: {\"verdicts\": [{\"id\": \"<rule id>\", \"verdict\": \"yes\"}]}\n
            Each verdict must be one of \"yes\", \"no\", or \"unsure\". Include exactly one entry
            for every rule id listed, copying each id character for character.'
        </code>
        </br>
        </br>
        But the plan is to use <a href=https://github.com/noamgat/lm-format-enforcer>lm-format-enforcer</a> to actually force the output to adhere to the pydantic model. Speaking of the pydantic model, it mirrors the input but has a verdict (yes/no/unsure) for each public_rule_id:
        </br>
        </br>
        <code>{
        
        '01a01b0f-2cf3-7110-8f89-9c59f02d2c51':'yes'\n
        '01a01b0f-2cf3-7110-8f89-9c59f02d2c52':'no'\n
        '01a01b0f-2cf3-7110-8f89-9c59f02d2c53':'unsure'
        
        }</code>
        
        See <a href='https://github.com/ataffe/MagPieCam-Core/blob/main/events/ml/prompt.py'>prompt.py</a> for reference.
        </br>
        </br>
        If a verdict is yes, a push notification is sent for the whole bundle along with a link to a preview image."

      # - title: "Hosting a model vs Using an API"
      #   content: "Coming Soon"
        
    
    images:
      - src: "/assets/images/projects/magpie/ios-rules.PNG"
        caption: "Example rules in the app."
      - src: "/assets/images/projects/magpie/ios-add-rule.PNG"
        caption: "Example of adding a rule."
    image_columns: 2
  section5:
    title: "Streaming Live Video"
    subsections:
      - title: "Starting 🎬"
        content: "I was surprised when developing this feature because at first, I thought streaming the video would be the most challenging part to build. But instead, <strong>starting and stopping streaming on-demand was actually the trickiest part</strong>. The camera always initiates connections with the backend because it sits in the user's wifi network. So I needed some way for the camera to know when to start / stop streaming. Otherwise, if the camera were streaming all the time that would waste bandwidth and compute. So I used long polling. Here is how it works from opposite ends.
        </br>
        </br>
        <u>The Camera</u>
        </br>
        The camera long polls the <code>cameras/streaming/command/</code> end point for 30 seconds at a time. If the command returned is none, or the camera is streaming and the command is start, the camera just calls the endpoint again. If the start command is received that camera starts a new process that consumes frame bytes from a buffer and feeds them to FFMPEG. FFMPEG then streams the frames to the MediaMTX server using RTSP including a JWT in the request. In the background the camera continues to long poll for new commands. If the stop command is received it signals the streamer process to end, and goes back to polling.
        </br>
        </br>
        <u>The iOS App</u>
        </br>
        When the user clicks on the live video view the iOS app starts by calling the <code>cameras/uuid:public_camera_id/streaming/start/</code> endpoint to signal the backend to publish a start command for the camera. Next the app starts reading the video stream from the MediaMTX via WebRTC / WHEP using a JWT for authentication. MediaMTX passes the token to the <code>cameras/mediamtx/auth/</code> in order to authenticate the stream. If the backend returns 200 OK then MediaMTX starts the stream.
       "
      - title: "Stopping 🛑"
        content: "To stop a video stream, the system uses Celery Beat to periodically run tasks that check the active streams. For each stream with an active publisher it checks if there are any readers for that stream. If there are no readers then it publishes the stop command, otherwise it does nothing. The channel subscriber then gets that message and the camera returns from long polling with the stop command. Once the camera returns from the long poll with the stop command it ends the RTSP stream."
    images:
      - src: "https://raw.githubusercontent.com/ataffe/MagPieCam-Assets/main/system_diagram/VideoStreamingDiagram.png"
        caption: "The video streaming and control diagram."
      - src: "https://raw.githubusercontent.com/ataffe/MagPieCam-Assets/main/system_diagram/VideoStreamingSeqDiagram.png"
        caption: "The sequence diagram for on-demand streaming.</br>(click to see full view)"
    image_columns: 1

---