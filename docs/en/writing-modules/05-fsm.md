
```python
from pydantic import BaseModel, Field, model_validator, ConfigDict
from mxc.fsm import StatesGroup, State
from mxc import utils
from .. import loader


class FeedbackFSM(StatesGroup):
    name = State()
    rating = State()
    comment = State()


class NamePayload(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    name: str = Field(min_length=2, max_length=50)

    @model_validator(mode='before')
    @classmethod
    def parse(cls, v):
        if isinstance(v, str):
            return {"name": v.strip()}
        return v


class RatingPayload(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    rating: int = Field(ge=1, le=5)

    @model_validator(mode='before')
    @classmethod
    def parse(cls, v):
        if isinstance(v, str):
            try:
                return {"rating": int(v.strip())}
            except ValueError:
                raise ValueError("Enter a number from 1 to 5")
        return v


@loader.tds
class FeedbackModule(loader.Module):
    strings = {
        "start": "📝 <b>Feedback</b><br><br>What is your name?",
        "ask_rating": "Nice to meet you, <b>{name}</b>!<br>Rate from 1 to 5:",
        "ask_comment": (
            "Thanks for the rating <b>{rating}/5</b>!<br><br>"
            "Write a comment (or send ? to skip):"
        ),
        "done": (
            "✅ <b>Done!</b><br><br>"
            "Name: <code>{name}</code><br>"
            "Rating: <b>{rating}/5</b><br>"
            "Comment: {comment}"
        ),
        "no_comment": "—",
    }

    @loader.command()
    async def feedback(self, mx, event):
        """Start the FSM feedback dialog"""
        mx.fsm.set_state(event, FeedbackFSM.name)
        await utils.answer(mx, self.strings["start"], event=event)

    @loader.state(FeedbackFSM.name)
    async def process_name(self, mx, event, ctx, payload: NamePayload):
        await ctx.update_data(name=payload.name)
        await ctx.set_state(FeedbackFSM.rating)
        await utils.answer(
            mx,
            self.strings["ask_rating"].format(name=payload.name),
            event=event,
        )

    @loader.state(FeedbackFSM.rating)
    async def process_rating(self, mx, event, ctx, payload: RatingPayload):
        await ctx.update_data(rating=payload.rating)
        await ctx.set_state(FeedbackFSM.comment)
        await utils.answer(
            mx,
            self.strings["ask_comment"].format(rating=payload.rating),
            event=event,
        )

    @loader.state(FeedbackFSM.comment)
    async def process_comment(self, mx, event, ctx):
        text = event.content.body.strip()
        comment = text if text != "?" else self.strings["no_comment"]
        data = await ctx.get_data()
        await ctx.clear()
        await utils.answer(
            mx,
            self.strings["done"].format(
                name=data.get("name", "?"),
                rating=data.get("rating", "?"),
                comment=comment,
            ),
            event=event,
        )
```

For more details on how FSM works: `../0-key-concepts.md` → section FSM
