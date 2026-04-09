# NestJS Coding Standards — Superbotics

Load `references/typescript.md` first. This file extends those rules with NestJS-specific patterns.

---

## Core Philosophy

NestJS applications at Superbotics follow a strict layered architecture:
**Controllers → Services → Repositories → Database**

No layer may skip another. Controllers never touch the database. Services never build HTTP responses. Repositories never contain business logic. This is non-negotiable.

---

## Directory Structure (NestJS)

```
src/
├── app.module.ts
├── main.ts
├── common/
│   ├── constants/
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   ├── middleware/
│   └── pipes/
├── config/
│   ├── app.config.ts
│   ├── database.config.ts
│   └── jwt.config.ts
├── database/
│   ├── migrations/
│   └── seeds/
└── modules/
    └── user/
        ├── user.module.ts
        ├── user.controller.ts
        ├── user.service.ts
        ├── user.repository.ts
        ├── user.entity.ts
        ├── dto/
        │   ├── create-user.dto.ts
        │   └── update-user.dto.ts
        └── interfaces/
            └── user.interface.ts

test/
└── modules/
    └── user/
        ├── user.controller.spec.ts
        ├── user.service.spec.ts
        └── user.repository.spec.ts
```

---

## File Naming Conventions

| File Type    | Convention                   | Example                       |
|--------------|------------------------------|-------------------------------|
| Module       | [name].module.ts             | user.module.ts                |
| Controller   | [name].controller.ts         | user.controller.ts            |
| Service      | [name].service.ts            | user.service.ts               |
| Repository   | [name].repository.ts         | user.repository.ts            |
| Entity       | [name].entity.ts             | user.entity.ts                |
| DTO          | [action]-[name].dto.ts       | create-user.dto.ts            |
| Interface    | [name].interface.ts          | user.interface.ts             |
| Guard        | [name].guard.ts              | jwt-auth.guard.ts             |
| Filter       | [name].filter.ts             | http-exception.filter.ts      |
| Interceptor  | [name].interceptor.ts        | logging.interceptor.ts        |
| Pipe         | [name].pipe.ts               | parse-uuid.pipe.ts            |
| Decorator    | [name].decorator.ts          | current-user.decorator.ts     |
| Config       | [name].config.ts             | database.config.ts            |
| Test         | [name].[type].spec.ts        | user.service.spec.ts          |

---

## Layered Architecture Rules

**Controllers → Services → Repositories → Database**

### Controllers
- HTTP layer only — receive request, call service, return response.
- Never write queries or business logic.
- Use @HttpCode() for explicit status codes.
- Use @Body(), @Param(), @Query() decorators exclusively.

### Services
- All business logic lives here.
- Call repositories for data access only.
- Throw NestJS HttpException subclasses (NotFoundException, ConflictException, etc.).
- Never build HTTP Response objects.
- Always decorated with @Injectable().

### Repositories
- All TypeORM database operations live here.
- Return null for not-found records — never throw HTTP exceptions.
- Never contain business logic.
- Always use @InjectRepository(Entity) pattern.
- Use softDelete() by default.

---

## Module Pattern

```typescript
@Module({
    imports: [TypeOrmModule.forFeature([UserEntity])],
    controllers: [UserController],
    providers: [UserService, UserRepository],
    exports: [UserService],
})
export class UserModule {}
```

---

## Controller Pattern

```typescript
@Controller('users')
export class UserController {
    public constructor(private readonly userService: UserService) {}

    @Post()
    @HttpCode(HttpStatus.CREATED)
    public async createUser(@Body() createUserDto: CreateUserDto): Promise<IUserProfile> {
        return this.userService.createNewUser(createUserDto);
    }

    @Get(':userId')
    public async getUserById(@Param('userId') userId: string): Promise<IUserProfile> {
        return this.userService.findUserById(userId);
    }
}
```

---

## Service Pattern

```typescript
@Injectable()
export class UserService {
    public constructor(private readonly userRepository: UserRepository) {}

    public async createNewUser(createUserDto: CreateUserDto): Promise<IUserProfile> {
        const existingUser = await this.userRepository.findByEmailAddress(createUserDto.emailAddress);

        if (existingUser !== null) {
            throw new ConflictException('An account with this email address already exists.');
        }

        return this.userRepository.createUserRecord(createUserDto);
    }
}
```

---

## Repository Pattern

```typescript
@Injectable()
export class UserRepository {
    public constructor(
        @InjectRepository(UserEntity)
        private readonly typeOrmRepository: Repository<UserEntity>,
    ) {}

    public async findById(userId: string): Promise<IUserProfile | null> {
        const foundUser = await this.typeOrmRepository.findOne({ where: { id: userId } });
        return foundUser ?? null;
    }

    public async deleteUserRecord(userId: string): Promise<void> {
        await this.typeOrmRepository.softDelete({ id: userId });
    }
}
```

---

## Entity Pattern

```typescript
@Entity({ name: 'users' })
export class UserEntity {
    @PrimaryGeneratedColumn('uuid')
    public id: string;

    @Column({ name: 'email_address', type: 'varchar', length: 255, unique: true })
    public emailAddress: string;

    @CreateDateColumn({ name: 'created_at' })
    public createdAt: Date;

    @DeleteDateColumn({ name: 'deleted_at', nullable: true })
    public deletedAt: Date | null;
}
```

---

## DTO Pattern

```typescript
export class CreateUserDto {
    @IsString()
    @IsNotEmpty()
    @MinLength(2)
    @MaxLength(255)
    public fullName: string;

    @IsEmail()
    @IsNotEmpty()
    public emailAddress: string;
}

// Update DTOs always extend with PartialType
export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

---

## NestJS Naming Quick Reference

| Construct         | Convention              | Example                   |
|-------------------|-------------------------|---------------------------|
| Module class      | PascalCase              | UserModule                |
| Controller class  | PascalCase              | UserController            |
| Service class     | PascalCase              | UserService               |
| Repository class  | PascalCase              | UserRepository            |
| Entity class      | PascalCase + Entity     | UserEntity                |
| DTO class         | PascalCase + Dto        | CreateUserDto             |
| Guard class       | PascalCase + Guard      | JwtAuthGuard              |
| Filter class      | PascalCase + Filter     | HttpExceptionFilter       |
| Interceptor class | PascalCase + Interceptor| LoggingInterceptor        |
| Interface         | PascalCase + I prefix   | IUserProfile              |
| Config token      | camelCase               | databaseConfig            |
| DB table          | snake_case plural       | order_items               |
| DB column         | snake_case              | created_at                |

---

## tsconfig.json Requirements

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "strictNullChecks": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "target": "ES2020",
    "module": "CommonJS",
    "moduleResolution": "node"
  }
}
```

---

## Forbidden Patterns

- No `process.env` in services, controllers, or repositories — use config module.
- No `console.log` in production — use NestJS `Logger`.
- No raw SQL in services or controllers — all queries through repository.
- No business logic in controllers.
- No HTTP exceptions in repositories — return `null`; let the service throw.
- No `any` types — use interfaces or `unknown` with narrowing.
- No skipping the global `ValidationPipe`.
- No circular module imports.
- No hardcoded secrets or API keys.
- No `TypeORM synchronize: true` in production — always use migrations.
- No `@ts-ignore` without a written reason and removal plan.
