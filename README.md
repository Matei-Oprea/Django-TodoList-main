
Proiect Python – To Do List (Interfata Django)


Proiectul are la baza creearea unei solutii de TO DO List care sa ii ajute pe utilizatori sa isi tina evidenta task-urilor.

Interfata web le permite mai multor utilizatori sa se logheze si sa isi acceseze task-urile, cat si numarul de task-uri ramase de completat. In cazul in care se doreste renuntarea la un anumit task acesta poate fi sters ulterior.

Pentru orice task este inclusa si o descriere ce are rolul de a detalia munca pe care utilizatorul trebuie sa o efectueze, iar task-urile pot fi cautarea prin intermediul search bar-ului integrat in partea de sus. Aplicatia va contoriza numarul de task-uri ramase de completat.

Avand in vedere ca am creat interfata in Django primul pas a fost acela de a intelege cum functioneaza framework-ul si ce contine un proiect nou din punct de vedere al fisierelor. 

Structura generala proiect in Django:

Cum se ruleaza programul:

•	Pentru inceput trebuie sa instalam o versiune de Python pe calculator -> https://www.python.org/

•	Urmatorul pas este instalarea pip package installer, care este un tool utilizat pentru instalarea librariilor si software-ului scris in Python. In general este deja instalat by default.

•	Mai intai verificam daca exista deja instalat:

pip –version

•	Pentru a upgrada pip-ul la ultima versiune putem folosi comanda:

pip install --upgrade pip


•	Urmatorul pas este sa instalam o versiune de Django:

pip install django

Daca nu avem VS Code instalat il putem instala de aici:
https://code.visualstudio.com/download

Urmatorul pas este acela de a instala extensiile necesare din VS Code anume:

-	Python
-	SqLite Viewer (optional pentru vizualizarea bazei de date)

Deschidem VS Code, iar la Files -> Open Folder selectam folderul proiectului care trebuie sa contina toate fisierele corespunzatoare.

Deschidem terminalul si pentru a rula programul introducem comanda:

python manage.py runserver


Accesam URL-ul serverului si putem sa utilizam aplicatia creata.


Descriere functionala:

Aplicatia este bazata pe 2 ramuri principale, partea de frontend si cea de backend.



Partea de frontend este regasita in subfolderul: Templates 
Functiile de  backend este sunt regasite in views.py, iar url-urile in urls.py.
 

Exemplu de functionare:

Cand este deschisa aplicatia prima pagina contine login-ul care se regaseste in Templates-> login.html. Cand dau click pe butonul Login se executa clasa  CustomLoginView(LoginView) din views.py si logheaza utilizatorul trimitandu-l la pagina task_list.html. 

In task_list.html sunt afisate dinamic din baza de date db.sqlite3 (baza de date integrata in framework-ul python) task-urile fiecarui utilizator. 
 
Daca nu exista niciun task existent se va afisa textul Create a New Task. 

Cand dam click pe New Task se apeleaza TaskCreate din views.py prin intermediul urls.py. 

Aici se creaza un nou task care are campurile: ‘title’, ‘’description’ si ‘complete’ in baza de date. Totodata, are loc si o verificare a user-ului pentru a testa daca este logat sau nu pentru a executa actiunea respectiva.
class TaskCreate(LoginRequiredMixin , CreateView):
    model = Task
    fields = ['title' , 'description' , 'complete']
    success_url = reverse_lazy('tasks')
    def form_valid(self, form): 
        form.instance.user = self.request.user  #check user
        return super(TaskCreate , self).form_valid(form)

Fiecare task poate fi completat sau sters, caz in care se elimina din lista (in partea de sus exista un contor care numara cate task-uri au ramas de solutionat), acesta regasindu-se in clasa TaskList din views.py: 
ontext ['count'] = context ['tasks'].filter(complete=False).count()

	Un alt element de mentionat este search bar-ul care in momentul cand scriem un input si apsam butonul search se va utiliza clasa TaskList si se va executa if-ul:
	if search_input:
            context['tasks'] = context['tasks'].filter(title__startswith=search_input)
        

 	In cazul in care dorim sa dam update la titlul / descrierea task-ului sau la statusul de completare

Practic aplicatia leaga partea de frontend cu cea de backend prin urls.py. din folderul Base.
from django.urls import path
from .views import TaskList, TaskDetail , TaskCreate , TaskUpdate, DeleteView , CustomLoginView , RegisterPage
from django.contrib.auth.views import LogoutView

urlpatterns = [
    path('login/' , CustomLoginView.as_view() , name='login'),
    path('logout/' , LogoutView.as_view(next_page='login') , name='logout'),
    path('register/', RegisterPage.as_view(), name='register'),
    path('', TaskList.as_view(), name='tasks'),
    path('task/<int:pk>/', TaskDetail.as_view(), name='task'),
    path('task-create/', TaskCreate.as_view(), name='task-create'),
    path('task-update/<int:pk>/', TaskUpdate.as_view(), name='task-update'),
    path('task-delete/<int:pk>/', DeleteView.as_view(), name='task-delete'),
    
]


In Base -> models.py avem modelul bazei de date cu campurile pe care le completam pentru fiecare task atat vizibile cat si cele din partea de backend pentru stocare (necesare by default):

class Task(models.Model):
    user = models.ForeignKey(User , on_delete=models.CASCADE , null=True , blank = True)
    title = models.CharField(max_length=200)
    description = models.TextField(null=True, blank=True)
    complete = models.BooleanField(default=False)
    create = models.DateTimeField(auto_now_add=True)
 	def __str__(self):
        return self.title
    class Meta:
        ordering = ['complete']





