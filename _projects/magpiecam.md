---
layout: project
title: "MagPieCam"
tagline: 'A smart camera system that enables users to define the notifications they want in plain English using the <a href="https://deepmind.google/models/gemma/gemma-4/">Gemma4</a> / <a href="https://ai.google.dev/gemini-api/docs/models/gemini-3.1-flash-lite">Gemini 3.1 Flash-Lite</a> and <a href="https://docs.ultralytics.com/tasks/detect">Yolo</a>.'
image: "/assets/images/projects/magpie/magpicam.jpg"
technologies:
  - "Python"
  - "Django"
  - "Computer Vision"
  - "Gemma4"
  - "ByteTrack"
  - "YOLOv11"
  - "PostgreSQL"
  - "Terraform"
  - "RESTful API"
  - "AWS"
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
   - "The MagPiCam iOS app consumes the MagPieCam-Core's REST API enabling users can manage rules and notifications"
   - "Live video is streamed from the MediaMTX server using WebRTC"
  backend:
    - "Django REST frameworks handles authentication and CRUD for users, cameras, rules, and notifications via a REST API secured with JWTs."
    - "Models are stored in a Postgres DB and use UUIDv7 ids as primary keys for sortable, opaque, index-friendly IDs"
    - "The event processor (a Django app) consumes SQS messages, and passes the payload to a Celery Worker"
    - "Celery workers evaluate images against user rules using the rules engine and triggers notifications."
    - "Cameras have their own JWT based a revokable device id derived from the Raspberry Pi cpu serial number and a UUID"
    - "The core services and DB's run in AWS using Fargate, RDS PostgreSQL, and ElastiCache Redis"
  edge agent:
    - "Tracks objects using ByteTrack and sends images to the backend with a per track exponential backoff"
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
motivation: I have two cats at home named Zia and Luna! They are 3 years old and very curious, so I need to keep an eye on them sometimes. Luna for example, has some health problems, and so sometimes I need to track when she is eating or using the bathroom. So I tried setting up a Ring cam near their litter box or food bowls for example. <strong> But I get many notifications for events that are not what I am looking for</strong>. Every time either cat or me for example, walk by I get an event. Additionally my girlfriend and I live in an area that has a decent amount of foot traffic. Similarly we tend to get a lot of notifications that are just people walking by. Ring now has unusual event detection but the <strong>user doesn't determine what is unusual.</strong> So I built MagPieCam.  
intro: MagPieCam lets users choose what they want to be notified about using rules they specify in natural language. The camera uses an object tracker (<a href="https://github.com/FoundationVision/ByteTrack">ByteTrack</a>) to recognize moving objects and then sends an images to the rules engine (Gemma4 / Gemini API) that decides if the image should trigger a push-notification. The camera itself is built using a Raspberry Pi Zero 2W and the Raspberry Pi AI Camera. On this page I will walk through the architecture of the system and some of the more interesting parts of building it.
note1: I used <a href="https://code.claude.com/docs/en/overview">Claude Code</a> throughout this project. I wanted to learn how to build a backend system with AI so I focused on the backend, and delegated the majority of the iOS and Edge development to Claude code. I talk more about my workflow below.
image1: "/assets/images/projects/magpie/zia_luna.jpg"
claude_workflow:
  intro: When I started this project, I wanted to solve a problem and learn about AI Engineering, by building a backend system that used an AI model. But I didn't' want to build just a backend. If I built just the backend, it would be very hard to conceptualize some of the interesting problems that I encountered while connecting the iOS app to the edge device, like streaming lifecycle for example. So I wrote the MagPieCam-Core services myself and delegated the a lot of the iOS app development and even more of the edge agent development to Claude Code. When designing the system, I would come up with a design myself and have Claude play systems architect and review for anything I was missing.
  practices:
    - title: "Backend by hand"
      description: "I usually started with a problem I want to solve, then would design a feature to solve it and then review it with Claude. When reviewing with Claude, I would usually ask clarifying questions and how engineering team at existing companies have solved the problem."
    - title: "Delegated iOS & Edge Agent"
      description: " When developing the iOS app, I would usually add a new REST endpoint, empty view, or view model and describe to Claude what I wanted to implement. Then either I would teak the view by hand or I would ask Claude tweak the design (mostly Claude) until it looked good."
  claude_image: "/assets/images/projects/magpie/claude-code-128px.png"
  takeaway: "I used to be very skeptical about delegating work to agents like Claude Code, but it really accelerated the development of this project. So far I have noticed that <strong>most mistakes come from miscommunications</strong>, so I try to be as specific as possible when working with Claude even if I don't know how to do what I want. Developing the app is a great case study of this. </br></br>I started by learning Swift using Stanford's <a href='https://cs193p.stanford.edu/'>cs193p course</a>. Then I started building the app. At first I built the views myself, but I am new to UI design and as views got more complex it was taking too much of my time. Swift has a vast library of tools for building views, so to save time I focused learning from other UIs so I could describe what I want and let Claude focus on how to implement that using best practices."
architecture:
  overview: "I think the most digestible way to look at the system is by looking at it feature by feature in the order that a user might encounter each feature."
  section1:
    title: "Creating an Account"
    content: "User account are pretty straightforward and implemented using Django's User model with a few fields added like Apple's push notification id for example. When a user creates an account, the app receives a short lived access token and a refresh token that can be used to retrieve a new access token. All other endpoints besides the device provision ones require JWTs."
    image1: "/assets/images/projects/magpie/ios-login.PNG"
    images_position: "right"
  section2:
    title: "Adding a camera"
    content: >-
      First to create a camera in the system I create a QR code using a
      <strong>claim token</strong> which is generated by the cameras/provision
      endpoint. The 'cameras/provision' endpoint takes a device id (CPU serial
      number on the Raspberry Pi) and generates a claim token by appending a
      uuid to the device id. The claim token is then hashed using SHA-256,
      stored in a Camera model in Postgres along with the device id. Then the original token is
      returned to the caller. This process basically simulates what would usually take
      place when the cameras are assembled. The claim token can only be used once
      to pair a camera to a user's app. This design ensures that if the camera is stolen it 
      can't be claimed. But before the user can claim the camera it must register 
      with the backend at start up.


      At startup, the camera calls the register endpoint passing it's device id and the claim_token.
      The backend core then queries the DB for the device id and compares the claim token (hash as well).
      If a matching camera is found in the DB, a flag is set on the camera model and another uuid is generated and
      attached to the device id, creating the <strong>device token</strong>. This
      token is saved to disk in the camera and is used to request JWTs going forward.
      This system enables the device token to be revoked in the case that the
      camera is lost or stolen.
    image1: "/assets/diagrams/magpie/DeviceProvisioningSeqDiagram.png"
    image2: "/assets/images/projects/magpie/ios-qrcode.PNG"
    images_position: "right"
  section3:
    title: "Creating a Rule"
    content: "coming soon"
  section4:
    title: "Streaming Live Video"
    content: "coming soon"
  section5:
    title: "Notifications"
    content: "coming soon"
---