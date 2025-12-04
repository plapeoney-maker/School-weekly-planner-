<html School weekly planner>
{
  "name": "school-weekly-planner",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev --name init",
    "prisma:seed": "ts-node prisma/seed.ts"
  },
  "dependencies": {
    "@prisma/client": "^5.0.0",
    "bcrypt": "^5.1.0",
    "body-parser": "^1.20.2",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.0",
    "node-cron": "^3.0.2"
  },
  "devDependencies": {
    "prisma": "^5.0.0",
    "ts-node": "^10.9.1",
    "ts-node-dev": "^2.0.0",
    "typescript": "^5.0.4"
  }
}
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
DATABASE_URL="file:./dev.db"
JWT_SECRET="replace-with-a-secure-secret"
PORT=4000
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

enum Role {
  ADMIN
  TEACHER
  OPS
}

enum ActivityType {
  MEETING
  SPORTS
  ACADEMIC
  CEREMONY
  OTHER
}

enum DutyStatus {
  PENDING
  CONFIRMED
  COMPLETED
  REJECTED
}

enum NotificationType {
  REMINDER
  ANNOUNCEMENT
  DUTY
  DRESS_CODE
  SYSTEM
}

model User {
  id                    String   @id @default(uuid())
  role                  Role
  name                  String
  email                 String   @unique
  phone                 String?
  avatarUrl             String?
  department            String?
  passwordHash          String
  notificationSettings  Json?
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt

  activities            ActivityParticipant[]
  dutiesAssigned        Duty[]   @relation("AssignedTo")
  dutiesCreated         Duty[]   @relation("AssignedBy")
  announcementsCreated  Announcement[] @relation("AnnouncementAuthor")
  notifications         Notification[]
}

model Activity {
  id           String   @id @default(uuid())
  title        String
  description  String?
  type         ActivityType
  startAt      DateTime
  endAt        DateTime
  location     String?
  dressCodeId  String?
  createdById  String
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
  attachments  Json?
  requiredItems Json?

  createdBy    User     @relation(fields: [createdById], references: [id])
  participants ActivityParticipant[]
  dressCode    DressCode? @relation(fields: [dressCodeId], references: [id])
  duties       Duty[]    @relation("ActivityDuties")
}

model ActivityParticipant {
  id               String  @id @default(uuid())
  activityId       String
  userId           String
  roleInActivity   String
  assignedAt       DateTime @default(now())
  status           String
  evidenceUrl      String?
  note             String?

  activity         Activity @relation(fields:[activityId], references:[id])
  user             User     @relation(fields:[userId], references:[id])
}

model Duty {
  id              String   @id @default(uuid())
  title           String
  description     String?
  startAt         DateTime
  endAt           DateTime
  assignedToId    String
  assignedById    String
  linkedActivityId String?
  reminderOffsets Json?
  status          DutyStatus @default(PENDING)
  completedAt     DateTime?
  evidence        Json?
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  assignedTo      User     @relation("AssignedTo", fields:[assignedToId], references:[id])
  assignedBy      User     @relation("AssignedBy", fields:[assignedById], references:[id])
  linkedActivity  Activity? @relation("ActivityDuties", fields:[linkedActivityId], references:[id])
}

model DressCode {
  id            String   @id @default(uuid())
  name          String
  icon          String?
  color         String?
  description   String?
  effectiveFrom DateTime?
  effectiveTo   DateTime?
  policyDocUrl  String?
  activities    Activity[]
}

model Announcement {
  id          String   @id @default(uuid())
  title       String
  body        String
  authorId    String
  pinned      Boolean  @default(false)
  priority    String
  visibility  String
  createdAt   DateTime @default(now())
  expiresAt   DateTime?
  attachments Json?
  author      User     @relation("AnnouncementAuthor", fields:[authorId], references:[id])
}

model Notification {
  id         String           @id @default(uuid())
  userId     String
  type       NotificationType
  payload    Json
  deliverAt  DateTime
  readAt     DateTime?
  channel    String
  createdAt  DateTime         @default(now())

  user       User             @relation(fields:[userId], references:[id])
}

model AuditLog {
  id         String   @id @default(uuid())
  actorId    String?
  actionType String
  objectType String
  objectId   String?
  metadata   Json?
  createdAt  DateTime @default(now())
}
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();
export default prisma;
import { Request, Response, NextFunction } from "express";
import jwt from "jsonwebtoken";
import dotenv from "dotenv";
import prisma from "../prismaClient";
dotenv.config();

const JWT_SECRET = process.env.JWT_SECRET || "secret-placeholder";

export interface AuthRequest extends Request {
  user?: any;
}

export async function authMiddleware(req: AuthRequest, res: Response, next: NextFunction) {
  const auth = req.headers.authorization;
  if (!auth) return res.status(401).json({ error: "Missing Authorization header" });
  const parts = auth.split(" ");
  if (parts.length !== 2 || parts[0] !== "Bearer") return res.status(401).json({ error: "Invalid Authorization format" });
  try {
    const payload: any = jwt.verify(parts[1], JWT_SECRET);
    const user = await prisma.user.findUnique({ where: { id: payload.sub } });
    if (!user) return res.status(401).json({ error: "User not found" });
    req.user = user;
    next();
  } catch (err) {
    return res.status(401).json({ error: "Invalid token" });
  }
}

export function requireRole(role: "ADMIN" | "TEACHER" | "OPS") {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user) return res.status(401).json({ error: "Not authenticated" });
    if (req.user.role !== role && req.user.role !== "ADMIN") {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
}
import express from "express";
import bodyParser from "body-parser";
import cors from "cors";
import dotenv from "dotenv";
import authRoutes from "./routes/auth";
import activitiesRoutes from "./routes/activities";
import dutiesRoutes from "./routes/duties";
import announcementsRoutes from "./routes/announcements";
import dressCodeRoutes from "./routes/dressCodes";
import notificationsRoutes from "./routes/notifications";
import reminderWorker from "./jobs/reminderWorker";

dotenv.config();

const app = express();
app.use(cors());
app.use(bodyParser.json());

app.use("/auth", authRoutes);
app.use("/activities", activitiesRoutes);
app.use("/duties", dutiesRoutes);
app.use("/announcements", announcementsRoutes);
app.use("/dress-codes", dressCodeRoutes);
app.use("/notifications", notificationsRoutes);

const PORT = Number(process.env.PORT || 4000);
app.listen(PORT, async () => {
  console.log(`Server started on http://localhost:${PORT}`);
  // start background worker
  reminderWorker.start();
});

