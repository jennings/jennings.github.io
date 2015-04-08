---
title: Repositories Aren't So Bad
layout: post
tags:
- entityframework
- patterns
---

Rob Conery has apparently been messing with .NET data access methods and
[doesn't much care for the Repository pattern][conery-repositories].

I agree with the assertion that the repository, as he presented it, is a bad
idea. However, rather than using CQRS or putting a `DbContext` directly on MVC
controllers, I think repositories can be done correctly.

[I made a repository on GitHub][dataaccess] that demonstrates three different
data access patterns.

## Using the EF context from the presentation/service layer

The first pattern (labeled "EFContext") uses the `DbContext` directly in the
service layer.

This is not a bad pattern at all, but requires the presentation layer to
reference `EntityFramework.dll`. In a solution that puts the data access layer
in a different assembly than the presentation layer, this seems wrong to me.

## Creating a "service repository"

I don't know if this pattern has a name, but one pattern I've seen is hiding
away the `DbContext` behind a so-called "repository". This is demonstrated in
the "ServiceRepository" projects.

The problem is that business logic tends to get shoved down into the data
access layer:

`StudentService` wants to update a collection of students. It can call the
`Update(student)` method on the repository:

    public void PromoteAllClasses()
    {
        var eligibleStudents =
            this.repository.GetAllStudents()
            .Where(s => !s.IsGraduated)
            .ToList();   // We must .ToList() it so we don't barf
                         // when Update() calls SaveChanges() in the
                         // middle of iterating

        // Note that this isn't an atomic operation
        foreach (var student in eligibleStudents)
        {
            // Here's some business logic
            student.YearsCompleted += 1;
            if (student.YearsCompleted >= 4)
                student.IsGraduated = true;

            this.repository.Update(student);
        }
    }

But with this pattern, each student update is its own transaction, and if the
graduation fails halfway through, then half your students have graduated and
half haven't. Oops.

If you instead move the "graudate all students" logic into the repository
layer, it can be an atomic operation:


        //// StudentService.cs

        public void PromoteAllClassesAtomic()
        {
            this.repository.PromoteAllClassesAtomic();
        }


        //// StudentServiceRepository.cs

        public void PromoteAllClassesAtomic()
        {
            var eligibleStudents = this.context.Students.Where(s => !s.IsGraduated);

            foreach (var student in eligibleStudents)
            {
                // Business logic in the repository?
                student.YearsCompleted += 1;
                if (student.YearsCompleted >= 4)
                    student.IsGraduated = true;
            }

            this.context.SaveChanges();
        }

But, now your data access layer contains both data access logic and business
logic. We're violating the Single Responsibility Principle. Why does the
service layer at this point exist, if all it does is ferry method calls to the
repository?

I don't know if this style of repository has a better name, but I don't think
it really represents a repository. It contains business logic, but it also
contains data access. It's a service, but also a repository. I wonder if the
name "Suppository" will stick.


## IUnitOfWork

This is the pattern I prefer to use. It keeps business logic in the service
layer but avoids referencing EF everywhere.

Our `StudentRepository` gains a new interface:

    public interface IStudentUnitOfWork
    {
        IRepository<Student> Students { get; }
        int SaveChanges();
    }

    internal class StudentContext : DbContext, IStudentUnitOfWork
    {
        public IRepository<Student> Students { get; private set; }

        public StudentContext()
        {
            this.Students = new Repository<Student>(this);
        }

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            // Map our entities by hand since we have no IDbSet<TEntity> properties
            modelBuilder.Entity<Student>().ToTable("Students");
        }
    }

The `SaveChanges()` method is implemented by `DbContext`, and we implement the
`IRepository` interface with a `Repository` object.

Unlike the Service Repository, this Repository has no business logic in it. It
exists only because we can't create a `DbSet<T>` ourselves (the constructor is
internal).

This is my preferred pattern because:

* Every class either encapsulates business logic or infrastructure, but not
  both.
* It's highly testable.
* Entity Framework is only referenced by the data access assembly.

[conery-repositories]: http://www.wekeroad.com/2014/03/04/repositories-and-unitofwork-are-not-a-good-idea/
[dataaccess]: https://github.com/jennings/DotNetDataAccess
