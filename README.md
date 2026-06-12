const Koa = require('koa');
const Router = require('koa-router');
const bodyParser = require('koa-bodyparser');
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/library');

const Book = mongoose.model('Book', new mongoose.Schema({
  title: { type: String, required: true },
  author: String,
  pages: Number,
  isAvailable: { type: Boolean, default: true }
}));

const app = new Koa();
const router = new Router();
app.use(bodyParser());

router.post('/books', async ctx => {
  const {title, author, pages, isAvailable} = ctx.request.body;
  ctx.body = await new Book({title, author, pages, isAvailable}).save();
  ctx.status = 201;
});

router.get('/books', async ctx => ctx.body = await Book.find());

router.get('/books/:id', async ctx => {
  const b = await Book.findById(ctx.params.id);
  if (!b) { ctx.status = 404; ctx.body = {error: 'Not found'}; return; }
  ctx.body = b;
});

router.put('/books/:id', async ctx => {
  const b = await Book.findByIdAndUpdate(ctx.params.id, ctx.request.body, {new: true, runValidators: true});
  if (!b) { ctx.status = 404; ctx.body = {error: 'Not found'}; return; }
  ctx.body = b;
});

router.delete('/books/:id', async ctx => {
  const b = await Book.findByIdAndDelete(ctx.params.id);
  if (!b) { ctx.status = 404; ctx.body = {error: 'Not found'}; return; }
  ctx.body = {message: 'Deleted'};
});

router.get('/books/thick', async ctx => ctx.body = await Book.find({pages: {$gt: 300}}));

app.use(router.routes()).use(router.allowedMethods());
app.listen(3000);
