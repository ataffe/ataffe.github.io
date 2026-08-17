---
layout: project
title: "MagPieCam"
tagline: 'A smart camera system that enables users to define the notifications they want in plain English using the <a href="https://deepmind.google/models/gemma/gemma-4/">Gemma4</a> / <a href="https://ai.google.dev/gemini-api/docs">Gemini API</a> and <a href="https://docs.ultralytics.com/tasks/detect">Yolo</a>.'
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
architecture_image: "https://raw.githubusercontent.com/ataffe/MagPieCam-Assets/main/system_diagram/magpie-cam-system-diagram-core.png"
architecture_features:
  ios app: 
   - "The MagPiCam iOS app consumes the MagPieCam-Core's REST API enabling users can manage rules and notifications"
   - "Live video is streamed from the MediaMTX server using WebRTC"
  backend:
    - "Django REST frameworks handles authentication and CRUD for users, cameras, rules, and notifications via a REST API."
    - "The event processor (a Django app) consumes SQS messages, and passes the payload to a Celery Worker"
    - "Celery workers evaluate images against user rules and triggers notifications."
    - "The REST API is secured using JWTs, and edge cameras have their own JWT based on the Raspberry Pi cpu serial number"
    - "Models are stored in a Postgres DB and use UUIDv7 ids as primary keys for sortable, opaque, index-friendly IDs"
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
motivation: I have two cats at home named Zia and Luna! They are 3 years old and very curious, so I need to keep an eye on them sometimes. Luna for example, has some health problems and so sometimes I need to monitor when she is eating or using the bathroom. So I tried setting up a Ring cam near their litter box or food bowls for example. But I get many notifications for events that are not what I am looking for. Every time either cat or me for example, walk by I get an event. Additionally my girlfriend and I live in an area that has a decent amount of foot traffic. Similarly we tend to get a lot of notifications that are just people walking by. Ring now has unusual event detection but <strong>the user doesn't determine what is unusual.</strong> So I built MagPieCam.  
intro: MagPieCam lets users choose what they want to be notified about using rules they specify in natural language. The camera uses an object tracker (<a href="https://github.com/FoundationVision/ByteTrack">ByteTrack</a> & Yolo) to recognize moving objects and then sends an images to the rules engine (Gemma4 / Gemini API) that decides if the image should trigger a push-notification. The camera itself is built using a Raspberry Pi Zero 2W and the Raspberry Pi AI Camera. On this page I will walk through the architecture of the system and some of the more interesting parts of building it.
note1: I used <a href="https://code.claude.com/docs/en/overview">Claude Code</a> throughout this project. I wanted to learn how to build a backend system with AI so I focused on the backend, and delegated the majority of the iOS and Edge development to Claude code. I talk more about my workflow below.
image1: "/assets/images/projects/magpie/zia_luna.jpg"
claude_workflow:
  intro: When started this project, I wanted to solve a problem and learn about AI Engineering, by building a backend that includes an AI model. But I didn't' want to build just a backend, because I wouldn't have been challenged to design the backend the way it is without the iOS app and the edge device. So I wrote the MagPieCam-Core services myself and delegated the majority of the iOS app and the edge agent to Claude Code. But overall I would work with Claude when designing each feature in the system. I would come up with a design and have Claude review it for anything I might be missing.
  practices:
    - title: "Backend by hand"
      description: "I usually started with a problem I want to solve, then would design a feature to solve it and then review it with Claude."
    - title: "Delegated iOS"
      description: "I usually would adding a new REST endpoint, empty view or view model and describe to Claude what I wanted to implement using best practices, and either I or Claude tweak the design (mostly Claude) until it worked and looked good."
    - title: Delegated Edge Agent
      description: "My workflow was pretty similar to the iOS app but here I would describe high level features to Claude. One advantage here was Claude can reduce dependencies on external libraries by building it's own version, which in the past would be impractical in a time-constrained environment."
claude_image: "/assets/images/projects/magpie/claude-code-128px.png"
---
